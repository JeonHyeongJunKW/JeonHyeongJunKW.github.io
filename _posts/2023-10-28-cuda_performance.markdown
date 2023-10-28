---
layout: post
title: CUDA 성능 나타내기
date: 2023-10-28 19:50:23 +0900
category: CUDA
tags : [CUDA, performance]
comments: true
use_math: true
---

### 계산하기

flops : 예상되는 연산 횟수 나타낸 것 (보통 주요 사칙 연산의 횟수를 총 더한 것을 의미함)

1 GFlops = 1000 Flops : 1000번의 예상 연산 횟수

GFlops / sec : 초당 예산되는 연산 횟수를 나타낼 때 쓴다.

- 보통 높으면 초당 계산할 수 있는 횟수가 늘어나기 때문에, 클수록 좋다.
- 계산할 때, flops는 동일하게 고정해놓고서 걸린 시간을 비교하는 방식을 사용한다.

### memory 읽기

Byte : 몇 바이트를 쓰거나 읽었는가?

ex) float A +=B -> A = A + B / 4바이트를 2번 읽어서 4바이트에 적음. 총 12바이트 메모리 접근이 일어남

Memory Bandwidth : 초당 발생한 메모리 접근량
GBytes/sec : 초당 몇 기가 바이트의 접근이 일어났는지를 표시함.

### Memory bounded

- 계산자체가 어렵다기 보다는 계산에 사용횟수가 많아서 생기는 연산 과정등을 수식할 때 사용됨.
- 보통의 cuda kernel 연산의 목적은 이러한 경우에 적합함

