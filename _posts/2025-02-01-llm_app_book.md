---
title: LLM을 활용한 실전 AI 애플리케이션 개발을 읽으며
date: 2025-02-01 16:53:00 +0900
categories: [Book, AIBook]
tags: [book, ai, llm]
render_with_liquid: false
---

![book](https://contents.kyobobook.co.kr/sih/fit-in/458x0/pdt/9791189909703.jpg)

이 글은 제목에도 있듯이 위 책을 읽고 개인적으로 중요하다고 생각되는 내용을 다시 적어보거나 추가적으로 AI에게 질문하거나 검색하여 정리한 글이다.

## 1부 LLM 기초 뼈대 세우기

### 1장 LLM 지도

1. LLM은 딥러닝 기반의 언어 모델이다.
![image1](https://blog.kakaocdn.net/dn/bsjnbF/btsDrFKhtaV/Il9rVp1O8T55iPw20KF3Jk/img.png)

2. 트랜스포머 모델이 어떻게 보면 현재의 AI가 있게 만든 주역이라 할 수 있다. 구글 연구원 8명이 2017년에 발표한 논문에서 발표되었는데,
현재 그 8명은 현 AI를 이끌고 있는 리더들이 되었다. 자세한 건 트랜스포머 관련 나무위키 문서를 참고하면 좋다. 
https://namu.wiki/w/%ED%8A%B8%EB%9E%9C%EC%8A%A4%ED%8F%AC%EB%A8%B8(%EC%9D%B8%EA%B3%B5%EC%8B%A0%EA%B2%BD%EB%A7%9D)

3. 딥러닝과 머신러닝의 차이는 다음과 같다. 머신러닝에 딥러닝이 포함되는 관계라고 볼 수 있는데, 자세히 살펴보면 큰 차이점이 있다.
![image2](https://blog.kakaocdn.net/dn/PFnjS/btsjlmxegd2/g2WNDKKUeKbIHzkRtC3h7k/img.png)
그림과 같이 데이터의 특징을 누가 뽑느냐에 따라 달라진다. 머신러닝은 데이터의 feature를 연구자 혹은 개발자가 찾고 모델에 입력으로 넣어
결과를 출력하는 반면, 딥러닝은 모델이 스스로 데이터의 feature를 찾는다.

4. 'Transfer Learning(전이 학습)'은 한 작업 또는 데이터 세트를 통해 얻은 지식을 사용하여 다른 관련 작업 및/또는 다른 데이터 세트에서 모델 성능을 개선하는 머신 러닝 기술이다. 전이 학습에서는 대량의 데이터로 모델을 학습시키는 Pre-training(사전 학습)과 특정한 문제를 해결하기 위한 데이터로 추가학습하는 Fine-Tuning(미세 조정)의 두 단계로 나눠 학습을 진행한다. 
![image3](https://cdn.botpenguin.com/assets/website/Transfer_Learning_c81a3a3957.png)

5. RNN과 트랜스포머를 비교하기 앞서 RNN을 살펴보면 다음과 같다. 

    + **RNN(Recurrent Neural Network, 순환 신경망)**은 시퀀스(sequence) 데이터를 처리하기 위해 설계된 딥러닝 모델입니다. 시퀀스 데이터란 시간 순서나 순차적인 구조를 가진 데이터를 의미하며, 예를 들어 텍스트, 음성, 시계열 데이터 등이 있습니다. RNN은 이러한 데이터를 처리할 때 이전 시간 단계의 정보를 기억하고 활용할 수 있는 특징을 가지고 있습니다.
    ![rnn](https://miro.medium.com/v2/resize:fit:1400/0*V5Q5gGhiDGurHd-z.png)

    대체로 RNN은 텍스트를 순차적으로 처리하여 다음 단어를 예측한다. 모델이 하나의 hidden state에 지금까지의 입력 텍스트의 맥락을 압축한다. 이와 다르게 트랜스포머는 맥락을 모두 참조하는 Attention 연산을 사용한다. 트랜스포머는 연산량이 RNN보다 많이 필요하지만,
    성능이 좋고 병렬 처리를 통해 학습 속도를 높일 수 있어 현재 대부분의 LLM은 트랜스포머 아키텍처를 기반으로 하고 있다.


6. LLM이 생성하는 답변을 사용자의 요청의 의도에 맞추는 것을 Alignment(정렬)이라고 한다.

7. Supervised Fine-Tuning, 지도 미세 조정은 사전학습된 모델을 특정 작업에 맞게 추가로 학습시키는 과정입니다. 이때, 레이블이 지정된 데이터(입력-출력 쌍)를 사용해 모델을 지도 학습 방식으로 미세 조정합니다. 지도 미세조정은 정렬을 위한 핵심 학습 과정으로서, 언어 모델리으로 사전 학습한 언어 모델을 Instruction Dataset으로 추가 학습하는 것을 뜻한다. 이때 지시 데이터셋은 사용자가 요청 또는 지시한
사항과 그에 대한 적절한 응답을 정리한 데이터셋을 의미한다. OpenAI는 수많은 데이터 작업자를 고용해 LLM이 받을 만한 질문과 그에 맞는 답변을
작성하게 하였고 이를 통해 지도 미세 조정을 수행했다.

8. RLHF (Reinforcement Learning from Human Feedback)은 인간의 피드백을 활용해 강화 학습(Reinforcement Learning) 방식으로 모델을 조정하는 방법입니다. 모델의 출력이 인간의 선호나 목표에 더 잘 부합하도록 학습시킵니다. 
그 예로, OpenAI에서는 두 가지 답변 중 사용자가 더 선호하는 답변을 선택한 데이터셋을 구축했는데, 이를 preference dataset이라고 한다.
이 데이터셋으로 LLM의 답변을 평가하는 Reward model를 만들고 LLM이 더 높은 점수를 받도록 추가 학습을 하였다.

9. sLLM은 Small Language Model의 약자로, 대규모 언어 모델(Large Language Models, LLMs)에 비해 상대적으로 작은 규모의 언어 모델을 의미합니다. sLLM은 제한된 컴퓨팅 자원과 데이터로도 효율적으로 학습하고 실행할 수 있도록 설계된 모델입니다. 이는 대규모 모델의 복잡성과 자원 요구량을 줄이면서도 특정 작업에서 충분한 성능을 발휘할 수 있는 모델을 목표로 합니다. 이처럼 sLLM을 활용해 특정 도메인 데이터나 작업에서 좋은 성과를 얻을 수 있다.

10. LLM(Large Language Model) 기반 AI 에이전트는 대규모 언어 모델을 활용해 복잡한 작업을 자율적으로 수행하는 지능형 시스템입니다. 단순한 질의응답을 넘어 환경 인식, 계획 수립, 도구 활용 등을 통해 목표를 달성하는 것이 특징입니다.

### 2장 LLM의 중추, 트랜스포머 아키텍쳐 살펴보기

#### 트랜스포머 아키텍처란?
![image4](https://raw.githubusercontent.com/dvgodoy/dl-visuals/main/Transformers/full_transformer.png)
트랜스포머 아키텍처는 크게 인코더와 디코더로 나뉜다. 왼쪽 상자는 언어를 이해하는 역할의 인코더이고, 오른쪽 상자는 언어를 생성하는 역할을
하는 디코더이다. 공통적으로 입력을 Embedding 층을 통해 숫자 집합인 임베딩으로 변환하고 positional encoding 층에서 문장의 위치 정보를 더한다. 인코더에서는 layer normalization(층 정규화), multi-head attention, feed foward 층을 거치며 문장을 이해하고 그 결과를 디코더로 전달한다. 디코더에서는 인코더에서와 유사하게 층 정규화, 멀티 어텐션 연산을 수행하면서 크로스 어텐션 연산을 통해 인코더가 전달한 데이터를 출력과 함께 종합해서 피드 포워드 층을 거쳐 결과를 생성한다.

#### 텍스트를 임베딩으로 변환

크게 세 가지 과정으로 구성되어 있다.

1. 텍스트를 적절한 단위로 잘라 숫자형 ID를 부여하는 **Tokenization**을 수행한다.

2. 토큰 아이디를 토큰 임베딩 층을 통해 여러 숫자의 집합인 토큰 임베딩으로 변환한다.

3. 위치 인코딩 층을 통해 토크의 위치 정보를 담고 있는 위치 임베딩을 추가해 최종적으로 모델에 입력할 임베딩을 만든다.

토큰화란 텍스트를 적절한 단위로 나누고 숫자 아이디를 부여하는 것을 말한다.
큰 단위를 기준으로 토큰화하면 텍스트의 의미는 잘 유지되나, 사전의 크기가 커진다.
단어를 기준으로 토큰화하는 경우, 이전에 본 적이 없는 새로운 단어는 사전에 없기 때문에 처리하지 못하는 OOV(Out Of Vocabulary)문제가
자주 발생한다.
반대로 작은 단위로 토큰화하면 사전의 크기가 작고 OOV 문제를 줄일 수 있으나 텍스트 의미가 잘 유지되지 않는다.

위치 인코딩의 경우, 절대적 위치 인코딩과 상대적 위치 인코딩이 있다. 이는 토큰의 고정된 위치를 인코딩하는지, 토큰 간의 상대적 거리로
인코딩하는지로 나뉜다.

#### 어텐션 이해하기

어텐션의 핵심 개념은 query, key, value라고 한다. 이는 정보 검색(Information Retrieval)에서 가져온 개념이다.

흔히 검색창에서 검색할 때, 입력하는 검색어를 'query'라고 한다. 쿼리를 입력하고 엔터 키를 치면, 검색 엔진은 수많은 자료 중에 관련 있는
문서를 찾는데 이때 쿼리와 관련이 있는지 계산하기 위해 문서가 가진 특징을 'key'라고 한다. 검색 엔진이 쿼리와 관련 깊은 키를 가진 문서를
찾아 관련도순으로 정렬해 문서를 제공할 때 문서를 'value'라고 할 수 있다.

![alt text](https://ffighting.net/wp-content/uploads/2023/05/image-23-1024x915.png)

트랜스포머 아키텍쳐에서는 토큰 임베딩을 변화하는 가중치 Wq,Wk,Wv를 도입하여 토큰과 토큰 사이의 관게를 계산하였다.

#### Attention vs Self-attention

+ Attention: 두 개의 서로 다른 시퀀스(또는 데이터) 간의 관계를 계산하는 메커니즘.

+ Self-Attention: 동일한 시퀀스 내의 요소들 간의 관계를 계산하는 메커니즘.

#### Multi-head Attention

한 번에 하나의 어텐션 연산보다 여러 어텐션 연산을 동시에 적용하는 것이 성능을 더 높인다.

![alt text](image-1.png)

#### Layer Normalization

트랜스포머(Transformer) 아키텍처에서 **층 정규화(Layer Normalization)**는 모델의 학습 안정성과 수렴 속도를 개선하기 위해 사용되는 중요한 기술입니다. 층 정규화는 주로 각 층의 출력을 정규화하여, 입력 데이터의 분포를 안정화시키고 그래디언트 소실(Gradient Vanishing) 또는 폭발(Gradient Explosion) 문제를 완화합니다.

#### Feed Forward Layer

피드 포워드 층은 데이터의 특징을 학습하는 Fully connected layer이다. 멀티 헤드 어텐션이 단어 사이의 관계를 파악하는 역할이라면, 피드
포워드 층은 입력 텍스트 전체를 이해하는 역할을 담당한다.

위 문장과 같이 책에 설명되어 있었는데 feed forward와 fully connected은 다른 개념이다. 모든 FC는 FF에 포함된다고 생각하면 편하다. 그 역은 성립하지 않는다.

Feed Forward Layer: 피드 포워드 층은 정보가 한 방향으로만 전달되는 구조를 가진 신경망의 기본 구성 요소입니다. 입력층에서 시작된 데이터가 은닉층을 거쳐 출력층으로 이동하며, 중간에 순환(recurrent)이나 피드백(feedback)이 없습니다.

![alt text](https://wikidocs.net/images/page/165070/back_propa_step_1.png)

Fully Connected Layer: 전결합층은 피드 포워드 층의 구체적인 형태로, 한 층의 모든 뉴런이 다음 층의 모든 뉴런과 연결된 구조를 말합니다. 이를 "완전 연결층" 또는 "Dense Layer"라고도 부릅니다.

![!\[alt text\](image-2.png)](https://builtin.com/sites/www.builtin.com/files/styles/ckeditor_optimize/public/inline-images/3_fully-connected-layer_0.jpg)

### 3장 트랜스포머 모델을 다루기 위한 허깅페이스 트랜스포머 라이브러리

사실 허깅페이스는 이번 과제를 진행하며 새로이 알게 되었다. 워낙 AI에 관심이 없었기도 했는데 요즘은 피할 수 없다. Huggingface는
트랜스포머나 데이터셋 같은 머신러닝 라이브러리를 제공하는 세계 최대의 인공지능 플랫폼 중 하나이다. 머신러닝계 깃허브로 불린다.

허깅페이스에서는 모델을 body와 head로 구분한다.

Deepseek에 물어보니 다음과 같은 답변이 나왔다.

```text
허깅페이스(Hugging Face)에서 모델의 **헤드(head)**와 **바디(body)**는 모델의 구조를 설명하는 용어입니다.

1. **바디(body)**:
   - 모델의 주요 부분으로, 대부분의 레이어와 파라미터가 포함됩니다.
   - 주로 사전 훈련된 모델(예: BERT, GPT, RoBERTa 등)을 의미하며, 입력 데이터를 처리하고 특징(feature)을 추출하는 역할을 합니다.
   - 바디는 일반적으로 특정 작업(예: 텍스트 분류, 질문 응답 등)에 구체적이지 않고, 범용적인 특징을 학습합니다.

2. **헤드(head)**:
   - 바디 위에 추가되는 특정 작업(task)을 위한 레이어입니다.
   - 예를 들어, 텍스트 분류를 위해 바디 위에 분류기(classification layer)를 추가하거나, 질문 응답을 위해 출력 레이어를 추가합니다.
   - 헤드는 일반적으로 작은 규모의 네트워크로, 특정 작업에 맞게 미세 조정(fine-tuning)됩니다.

예를 들어, BERT 모델을 텍스트 분류 작업에 사용할 경우:
- **바디**: BERT의 모든 레이어 (입력 텍스트를 처리하고 특징을 추출).
- **헤드**: 분류를 위한 추가 레이어 (예: fully connected layer + softmax).
```




















