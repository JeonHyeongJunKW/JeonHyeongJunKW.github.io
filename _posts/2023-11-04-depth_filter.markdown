---
layout: post
title: Directional Joint Bilateral Filter for Depth Images 리뷰
date: 2023-11-04 16:17:23 +0900
category: Preprocessing
tags : [Depth, Preprocessing]
comments: true
use_math: true
---

## 리뷰 이유
- Depth 정보의 불확실성을 인식하고 이해한다.
- Depth 필터링 기법과 특성을 알아본다.


## 기존 필터링 방식들
- 일반적인 depth에 대한 필터링 방법들은 엣지에 대해서 적응형방식을 사용한다.
- 보통은 bilateral filter와 median filter를 사용한다.
- 또한 일반적인 알고리즘은 전경(foreground)과 배경(background)를 분리하여, 유용한 픽셀 정보를 사용하고자한다. 하지만 이러한 방식은 계산복잡도도 높고, 분류결과도 그다지 신뢰하기 힘들다.

## 이 논문에서 제안하는 방법의 장점
- 제안 방법은 trilateral filter를 non-edge 영역에 적용하고, directional bilateral filter를 edge 영역에 적용한다.
  - trilateral filter는 노이즈한 non-edge영역에서 joint bilateral filter를 능가한다.
  - directional bilateral filter는 median filter에 비해서 depth edge를 좀더 선명하게 만들어준다.
- 제안 방법은 depth hole pixel과 같은 영역에 속하는 이웃 픽셀들이 구멍 채우기(hole filling)에 사용되는 방식으로 필터링이 적용된다.

## 제안방법
- non-hole(depth noise가 많은 지역)과 hole(depth가 없는 지역)을 각각 나눠서 필터링을 적용한다.

![components](/public/img/Preprocessing/Depth_preprocessing_1.png)

### 전처리

1. depth 5*5 closing 연산 (작은 depth 구멍을 채운다.)
2. Canny edge detector를 사용하여 color와 depth의 엣지를 찾는다.
3. color 이미지에 있는 엣지중에서 물체의 경계가 아닌 부분을 제거하기 위해서, 각 color 엣지의 7x7 픽셀 근처에 depth 엣지가 없는 color 엣지픽셀을 제거한다.

![components](/public/img/Preprocessing/Depth_preprocessing_2.png)

4. 이 결과물에서 7 x 7 윈도우 안에 edge depth 픽셀은 hole pixel로 간주된다. (그래서 depth 구멍을 확장한다.)

### non-hole 영역 필터링
- 노이즈가 많은 영역이라서 필터링이 필요함

#### 엣지가 아닌 영역에 대한 필터링
- 수정된 Joint Bilateral filter를 적용한다.

![components](/public/img/Preprocessing/Depth_preprocessing_3.png)
- non-hole에 대해서는 위의 인테그랄에서 적용하지 않는다.
- 인테그랄 내부의 4가지 곱은 각각 이웃 depth, spatial gaussian kernel, 이웃 픽셀의 이미지 밝기에 대한 gaussian kernel, 이웃 픽셀의 depth에 대한 가우시안 커널이다.
- 유사한 depth를 가지는 픽셀에 좀더 점수를 많이 주도록 설정함.

#### 엣지인 영역에 대한 필터링
- 위의 방법 대신에 기존 Joint Bilateral filter + 엣지의 방향 정보를 활용하여 엣지의 뚜렷함을 이용한다.
- 중간 항이 수정되며 directional gaussian filter를 사용한다.

![components](/public/img/Preprocessing/Depth_preprocessing_4.png)
- 엣지의 경계방향에 존재하는 이웃 depth에 대해서 좀더 가중치를 둔다.
- 수평, 수직방향 표준편차와 엣지 방향이 사용된다.
- 엣지의 방향은 atan2(g_y, g_x)로 계산된다.(해당 엣지 지점에서의 g_x, g_y는 공간상의 gradient이다.)
- 사실상 표준편차를 같은 비율 설정하면, 회전 자체는 의미가 없다. 아래의 가중치 사진 참조

![Alt text](/public/img/Preprocessing/Depth_preprocessing_5.png)
