---
title: Swin Transformer 코드 뜯어보기
date: 2025-04-18 00:40:00 +0900
categories: [ai]
tags: [ai, swin transformer]
render_with_liquid: false
---

시작하기 앞서 nn.module을 잠깐 살펴보면,

"nn.Module은 PyTorch에서 신경망 레이어와 모델의 기본 뼈대가 되는 클래스이며, 파라미터 관리, 연산 정의, 모델 저장/불러오기 등 딥러닝 개발을 위한 핵심 기능을 제공한다."

## Cyclicshift

```python
class Cyclicshift(nn.Module):
    """ cyclic shift """
    def __init__(self, displacement):
        super().__init__()
        self.displacement = displacement

    def forward(self, x):
        return torch.roll(x, shifts=(self.displacement, self.displacement), dims=(1, 2))
```

+ torch.roll: PyTorch의 텐서 순환 이동 함수

+ shifts=(displacement, displacement): 높이(height)와 너비(width) 방향으로 동일한 크기 이동

+ dims=(1, 2): (배치, 채널, 높이, 너비) 형태 입력 기준으로 높이(1), 너비(2) 차원 선택

이동 후 남는 영역은 반대쪽에서 채워지는 순환 패딩 효과가 있다.

```text
원본 행렬:
1  2  3  4
5  6  7  8
9 10 11 12
13 14 15 16

CyclicShift(displacement=2) 후:
11 12  9 10
15 16 13 14
3  4  1  2
7  8  5  6
```

## Residual

```python
class Residual(nn.Module):
    def __init__(self, fn):
        super().__init__()
        self.fn = fn

    def forward(self, x, **kwargs):
        return self.fn(x, **kwargs) + x
```


**Residual Connection**을 구현

## PreNorm

```python
class PreNorm(nn.Module):
    def __init__(self, dim, fn):
        super().__init__()
        self.norm = nn.LayerNorm(dim) # (1) 레이어 정규화
        self.fn = fn                  # (2) 적용할 함수

    def forward(self, x, **kwargs):
        return self.fn(self.norm(x), **kwargs) # (3) 정규화 → 함수 적용
```

## FeedForward

```python
class FeedForward(nn.Module):
    def __init__(self, dim, hidden_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(dim, hidden_dim),
            nn.GELU(),
            nn.Linear(hidden_dim, dim),
        )

    def forward(self, x):
        return self.net(x)
```

+ `nn.Linear(dim, hidden_dim)`: 
입력을 dim차원에서 hidden_dim차원으로 선형 변환(가중치 곱 + 바이어스)

+ `nn.GELU()`:
비선형 활성화 함수(Gaussian Error Linear Unit)

![image](https://pytorch.org/docs/stable/_images/GELU.png)

+ `nn.Linear(hidden_dim, dim)`:
다시 hidden_dim에서 원래 차원인 dim으로 선형 변환

## create_mask

```python
def create_mask(window_size, displacement, upper_lower, left_right):
    """ attention mask """
    mask = torch.zeros(window_size ** 2, window_size ** 2) # 마스크 초기화

    if upper_lower: # 아래->위 차단
        mask[-displacement * window_size:, :-displacement * window_size] = float('-inf')
        mask[:-displacement * window_size, -displacement * window_size:] = float('-inf')

    if left_right: # 위->아래 차단단
        mask = rearrange(mask, '(h1 w1) (h2 w2) -> h1 w1 h2 w2', h1=window_size, h2=window_size)
        mask[:, -displacement:, :, :-displacement] = float('-inf')
        mask[:, :-displacement, :, -displacement:] = float('-inf')
        mask = rearrange(mask, 'h1 w1 h2 w2 -> (h1 w1) (h2 w2)')

    return mask
```

create_mask 함수는 Swin Transformer의 Shifted Window 메커니즘에서 인접하지 않은 영역 간 어텐션을 차단하기 위한 마스크를 생성.

## get_relative_distances

```python
def get_relative_distances(window_size):
    """ for relative positional bias """
    indices = torch.tensor(np.array([[x, y] for x in range(window_size) for y in range(window_size)]))
    distances = indices[None, :, :] - indices[:, None, :]
    return distances
```

**Relative Positional Embedding**을 계산하기 위해 윈도우 내 각 위치 간 상대적 좌표 차이를 구하는 역할을 함.

1. 인덱스 생성 

2. 텐서 변환: 리스트를 넘파이 배열 → 파이토치 텐서로 변환

3. 상대 좌표 계산: 모든 좌표 쌍 간 차이 계산

## WindowAttention

```python
class WindowAttention(nn.Module):
    """ W-MSA and SW-MSA """
    def __init__(self, dim, heads, head_dim, shifted, window_size, relative_pos_embedding):
        super().__init__()
        inner_dim = head_dim * heads

        self.heads = heads
        self.scale = head_dim ** -0.5
        self.window_size = window_size
        self.relative_pos_embedding = relative_pos_embedding
        self.shifted = shifted

        # SW-MSA
        if self.shifted:
            displacement = window_size // 2
            self.cyclic_shift = Cyclicshift(-displacement)
            self.cyclic_back_shift = Cyclicshift(displacement)
            self.upper_lower_mask = nn.Parameter(
                create_mask(window_size=window_size, displacement=displacement, upper_lower=True, left_right=False),
                requires_grad=False,
            )
            self.left_right_mask = nn.Parameter(
                create_mask(window_size=window_size, displacement=displacement, upper_lower=False, left_right=True),
                requires_grad=False,
            )
        
        self.to_qkv = nn.Linear(dim, inner_dim * 3, bias=False)

        # relative position bias
        if self.relative_pos_embedding:
            self.relative_indices = get_relative_distances(window_size) + window_size - 1
            self.pos_embedding = nn.Parameter(torch.randn(2 * window_size - 1, 2 * window_size - 1))
        else:
            self.pos_embedding = nn.Parameter(torch.randn(window_size ** 2, window_size ** 2))

        self.to_out = nn.Linear(inner_dim, dim)

    def forward(self, x):
        # for cyclic shift (SW-MSA)
        if self.shifted:
            x = self.cyclic_shift(x)

        b, n_h, n_w, _ = x.shape
        h = self.heads

        qkv = self.to_qkv(x).chunk(3, dim=-1) # divide into three sub-tensors in the last dimension (dim=-1)
        nw_h = n_h // self.window_size
        nw_w = n_w // self.window_size

        q, k, v = map(
            lambda t: rearrange(
                t, 'b (nw_h w_h) (nw_w w_w) (h d) -> b h (nw_h nw_w) (w_h w_w) d',
                h=h, w_h=self.window_size, w_w=self.window_size,
            ),
            qkv,
        )

        dots = einsum('b h w i d, b h w j d -> b h w i j', q, k) * self.scale

        # for relative position bias
        if self.relative_pos_embedding:
            dots += self.pos_embedding[self.relative_indices[:, :, 0], self.relative_indices[:, :, 1]]
        else:
            dots += self.pos_embedding

        # for attention mask (SW-MSA)
        if self.shifted:
            dots[:, :, -nw_w:] += self.upper_lower_mask
            dots[:, :, nw_w-1::nw_w] += self.left_right_mask

        attn = dots.softmax(dim=-1)

        out = einsum('b h w i j, b h w j d -> b h w i d', attn, v)
        out = rearrange(out, 'b h (nw_h nw_w) (w_h w_w) d -> b (nw_h w_h) (nw_w w_w) (h d)',
                        h=h, w_h=self.window_size, w_w=self.window_size, nw_h=nw_h, nw_w=nw_w)
        out = self.to_out(out)

        # for reverse cyclic shift (SW-MSA)
        if self.shifted:
            out = self.cyclic_back_shift(out)

        return out
```

W-MSA와 SW-MSA를 구현하였다.

## SwinBlock

```python
class SwinBlock(nn.Module):
    """ one swin transformer block """
    def __init__(self, dim, heads, head_dim, mlp_dim, shifted, window_size, relative_pos_embedding):
        super().__init__()
        self.attention_block = Residual(PreNorm(dim, WindowAttention(dim=dim, heads=heads, head_dim=head_dim, 
                                                                     shifted=shifted, window_size=window_size, relative_pos_embedding=relative_pos_embedding)))
        self.mlp_block = Residual(PreNorm(dim, FeedForward(dim=dim, hidden_dim=mlp_dim)))

    def forward(self, x):
        x = self.attention_block(x)
        x = self.mlp_block(x)
        return x
```

Swin Trasformer에서 기본 block을 구현하였다.

`입력 특징맵 → [정규화 → 창어텐션 → 잔차] → [정규화 → MLP → 잔차] → 출력 특징맵`.

## PatchMerging

```python
class PatchMerging(nn.Module):
    """ patch merging """
    def __init__(self, in_channels, out_channels, downscaling_factor):
        super().__init__()
        self.downscaling_factor = downscaling_factor
        self.patch_merge = nn.Unfold(kernel_size=downscaling_factor, stride=downscaling_factor, padding=0)
        self.linear = nn.Linear(in_channels * downscaling_factor ** 2, out_channels)

    def forward(self, x):
        b, c, h, w = x.shape
        new_h, new_w = h // self.downscaling_factor, w // self.downscaling_factor
        x = self.patch_merge(x).view(b, -1, new_h, new_w).permute(0, 2, 3, 1)
        x = self.linear(x)
        return x
```

이미지 다운샘플링을 수행하며 Swin Transformer의 계층적 특징 추출을 가능하게 하는 핵심 모듈.

## StageModule

```python
class StageModule(nn.Module):
    """ one stage block (patch merging -> swin transformer block) """
    def __init__(self, in_channels, hidden_dimension, layers, downscaling_factor, num_heads, head_dim, window_size, relative_pos_embedding):
        super().__init__()
        assert layers % 2 == 0, 'Stage layers need to be divisible by 2 for regular and shifted block.'

        # patch merging
        self.patch_partition = PatchMerging(in_channels=in_channels, out_channels=hidden_dimension, downscaling_factor=downscaling_factor)

        # swin transformer block
        self.layers = nn.ModuleList([])
        for _ in range(layers // 2):
            self.layers.append(nn.ModuleList([
                SwinBlock(dim=hidden_dimension, heads=num_heads, head_dim=head_dim, mlp_dim=hidden_dimension * 4,
                          shifted=False, window_size=window_size, relative_pos_embedding=relative_pos_embedding), # W-MSA
                SwinBlock(dim=hidden_dimension, heads=num_heads, head_dim=head_dim, mlp_dim=hidden_dimension * 4,
                          shifted=True, window_size=window_size, relative_pos_embedding=relative_pos_embedding), # SW-MSA
            ]))

    def forward(self, x):
        x = self.patch_partition(x)
        for regular_block, shifted_block in self.layers:
            x = regular_block(x)
            x = shifted_block(x)
        return x.permute(0, 3, 1, 2)
```

생성자에서 볼 수 있듯이, patch merging과 swin transformer를 과정을 합친 하나의 stage module이다.

## SwinTransformer

```python
class SwinTransformer(nn.Module):
    def __init__(self, *, hidden_dim, layers, heads, channels=3, num_classes=1000, head_dim=32, window_size=7,
                 downscaling_factors=(4, 2, 2, 2), relative_pos_embedding=True, for_detection=False):
        super().__init__()

        self.for_detection = for_detection # for object detection (remove mlp head layer)

        self.stage1 = StageModule(in_channels=channels, hidden_dimension=hidden_dim, layers=layers[0],
                                  downscaling_factor=downscaling_factors[0], num_heads=heads[0], head_dim=head_dim,
                                  window_size=window_size, relative_pos_embedding=relative_pos_embedding)
        self.stage2 = StageModule(in_channels=hidden_dim, hidden_dimension=hidden_dim * 2, layers=layers[1],
                                  downscaling_factor=downscaling_factors[1], num_heads=heads[1], head_dim=head_dim,
                                  window_size=window_size, relative_pos_embedding=relative_pos_embedding)
        self.stage3 = StageModule(in_channels=hidden_dim * 2, hidden_dimension=hidden_dim * 4, layers=layers[2],
                                  downscaling_factor=downscaling_factors[2], num_heads=heads[2], head_dim=head_dim,
                                  window_size=window_size, relative_pos_embedding=relative_pos_embedding)
        self.stage4 = StageModule(in_channels=hidden_dim * 4, hidden_dimension=hidden_dim * 8, layers=layers[3],
                                  downscaling_factor=downscaling_factors[3], num_heads=heads[3], head_dim=head_dim,
                                  window_size=window_size, relative_pos_embedding=relative_pos_embedding)
        
        self.mlp_head = nn.Sequential(
            nn.LayerNorm(hidden_dim * 8),
            nn.Linear(hidden_dim * 8, num_classes)
        )

    def forward(self, img):
        x = self.stage1(img) # (B, 96, 56, 56) for swin_t
        x = self.stage2(x) # (B, 192, 28, 28) for swin_t
        x = self.stage3(x) # (B, 384, 14, 14) for swin_t
        x = self.stage4(x) # (B, 768, 7, 7) for swin_t

        if not self.for_detection:
            x = x.mean(dim=[2, 3]) # (B, 768) for swin_t
            return self.mlp_head(x) # (B, num_classes=1000)
        else:
            return x
```

위의 전 과정을 합친 하나의 SwinTransformer다.

![image](https://user-images.githubusercontent.com/26739999/142576715-14668c6b-5cb8-4de8-ac51-419fae773c90.png)