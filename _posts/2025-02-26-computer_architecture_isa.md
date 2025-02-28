---
title: 컴퓨터 아키텍처 - Instruction Set Architecture
date: 2025-02-26 14:23:00 +0900
categories: [Computer Science & Engineering, Computer Architecture]
tags: [computer architecture, instruction set architecture]
render_with_liquid: false
---

이번에 CS의 핵심 과목 중 하나인 '컴퓨터 구조'를 수강하며 개인적으로 기록을 해두려 한다. 오늘 포스트는 그 첫 단추인 ISA이다.

## Instruction Set Architecture vs Microarchitecture

Instruction Set Architecture(ISA)는 소프트웨어와 하드웨어, 특히 CPU와의 사이의 약속이다. ISA는 여러 명령어를 정의하며
현재 시스템의 상태가 어떻게 구성되어 있고 명령어를 실행할 때 그 상태가
어떻게 바뀌는지에 대해서 정의한다. 이 ISA를 통해 SW를 구현하는 프로그래머 입장에서는 작성한 프로그램이 컴퓨터에서 어떻게 실행될 것인지 알 수 있고, 하드웨어를 구현하는 설계자 입장에서 어떤 명령이 수행되었을 때 어떻게 수행되도록 설계해야 하는지의 명세서가 된다.

Microarchitecture는 CPU나 GPU 같은 하드웨어가 작동하는 방식을 서술한 일종의 컴퓨터 설계도이며, 따라서 하드웨어의 운영에 대해 세세하게 기술 되어 있다.

정리해보면, ISA는 프로세서가 *무엇*을 할 수 있는지 Interface를 정의하고 Microarchitecture는 그것을 *어떻게* 수행하는지(구현)을 정의한다.

## Von Neumann Architecture

폰 노이만이 제시했다고 알려진 컴퓨터 구조이다. 이론상 튜링 머신과 같은 일을 할 수 있어 Turing Complete하다. 

폰 노이만 구조의 디지털 컴퓨터에서는 'stored-program'의 개념이 도입되었다. 이는 프로그램을 구성하는 명렁어들을 임의 접근이 가능한 메모리상에 실행 순서대로 배열하고, 동시에 조건 분기를 무제한으로 허용한다는 것을 뜻한다. 폰 노이만 구조에서는 실행 코드와 데이터가 따로 구분되어 있지 않고, 모두 같은 메모리 속에 혼재되어 있다. 이를 분리하는 건 Operating System의 몫이다. 

폰 노이만 구조는 CPU, Memory, Program의 3가지 구성요소로 이루어져 있다.

Instruction에 집중하면 이들은 1차원 메모리에 저장되어 있고 데이터와 마찬가지로 수정가능하다. 

### vs Dataflow

![image](https://slideplayer.com/slide/15539653/93/images/29/von+Neumann+vs+Dataflow.jpg)

폰노이만 구조에서는 명렁어가 메모리에 저장된 순서대로 처리된다. Dynamic하지만, 이른바 '폰노이만 병목현상'이 발생한다. 단일 메모리 공간으로 데이터와 명령어를 전송하는 데 동일한 버스를 사용하기 때문이다. 반면, Dataflow에서는 그 의존성에 따라 결정된다. 이미지로 볼 수 있듯이, 위와 같이 처리한다면 병렬 처리를 통해 병목 현상을 피할 수 있다.

## General Instruction Classes

Arithmetic and Logical operations(e.g., add, sub, and, or): 일반적으로 integer와 floating-point 명령어로 나뉜다.

1. 특정 위치에서 operand를 불러온다.

2. operand를 계산한다.

3. 결과값을 특정 위치에 저장한다.

4. 다음 명렁어로 PC(program counter)를 업데이트한다.

Data movement operations(e.g., move, load, store):

1. 특정 위치에서 operand를 불러온다.

2. operand를 특정 위치에 저장한다.

3. 다음 명령어로 PC 업데이트.

Control flow operations(e.g., branch, jump)

1. 특정 위치에서 operand 불러온다.

2. 조건문과 target address를 계산한다.

3. 조건문이 참이면 target address, 아니면 다음 명령어로 이동.

## Atomicity of an Instruction

이 원자성은 DB에서도 언급되는 개념이었는데 여기서도 나왔다.

SW 관점에서, 모든 명령어는 항상 완전히 실행되거나 실행되지 않아야 한다.
즉, 명령어가 실행되는 도중에 발생할 수 있는 그 중간 상태는 프로그래머에게 절대 관찰되어서는 안 된다.

## Operand Sources

Q: Can ALU operands be in memory?

답은 둘 다이다.

Complex Instruction Set Computer(CISC)에서는 Instruction 길이가 다양하다. 그리고 operand가 메모리에 있다.

Reduced Instruction Set Computer(RISC)에서는 메모리에 저장될 수 없으며, Instruction 길이는 고정되어 있다. operand는 무조건 register에 와야 한다. 이는 *load-store architecture*라 불리며 여기서 load-store는 register라고 생각하면 편하다.

## Memory Addressing Modes

여기서 LD는 load, rt는 register를 의미한다. 참고로, RISC-V에서는 레지스터가 총 32개이다. GPR은 General Purpose Register이다.

`GPR[r_base]`는 r_base 레지스터의 값이다.

Absolute: `LD rt, 10000`

Register indirect: `LD rt, (r_base)` 

Displaced or based: `LD rt, offset(r_base)`. offset+`GPR[r_base]`.

Indexed: `LD rt, (r_base, r_index)`. `GPR[r_base]`+`GPR[r_index]`

Memory indirect: `LD rt, ((r_base))`. 여기서는 `GPR[r_base]`를 메모리의 주소로 사용하고 있다.

Auto inc/decrement..

## RISC(Reduced Instruction Set Computer)

RISC의 장점들을 살펴보자.

1. **Simple** operations: 2 input, 1 output의 arithmetic, logical operations

2. **Simple** data movements: ALU 연산은 register-to-register.
메모리는 오직 load와 store 명령어로만 접근 가능하다.(load-store architecture)

3. **Simple** branches: 제한된 조건문과 타겟의 다양성.

4. **Simple** instruction encoding: 모든 명령어들은 같은 수의 비트로 부호화되어 있다. 

이제는 CISC가 아닌 RISC의 시대.

## Inter-Model Compatibility

Inter-Model Compatibility는 컴퓨터 시스템의 다양한 구성(configuration) 간에 프로그램이 정상적으로 실행될 수 있도록 보장하는 개념이다. 이는 동일한 ISA(Instruction Set Architecture)를 사용하는 시스템 간의 소프트웨어 호환성을 유지하기 위해 중요한 요소이다.

*Strict program compatibility = “a valid program whose logic will not depend implicitly upon time of execution and which runs upon configuration A, will also run on configuration B if the latter includes at least the required storage, at least the required I/O devices ….”*

## RISC-V Programmar Visible State

![image](https://velog.velcdn.com/images/io0818/post/d4a66c03-235e-4a78-b0b2-0f96d5740660/image.png)

base register에서 x0는 항상 0임을 눈여겨봐야 한다. 이는 constant로서 0을 사용하기 위함이다. 레지스터는 총 32개이므로, 5bit로 표현가능하다.
사실 더 많은 레지스터가 있으면, 더 flexible할 수 있으나 더 작으면 더 빠르기 때문이다.



