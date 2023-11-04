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
3. color 이미지에 있는 엣지중에서 물체의 경계가 아닌 부분을 제거하기 위해서, 각 color 엣지의 7x7 픽셀 근처에 depth 엣지가 없는 color 픽셀은 제거된다.
4. 이웃 엣지 픽셀의 수가 임계값(T_n)보다 적은 엣지 픽셀은 제거된다.

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
- 위의 방법 대신에 기존 Joint Bilateral filter + 엣지의 방향 정보(directional)를 활용하여 엣지의 뚜렷함을 이용한다. (DJBF)
- 중간 항이 수정되며 directional gaussian filter를 사용한다.

![components](/public/img/Preprocessing/Depth_preprocessing_4.png)
- 엣지의 경계방향에 존재하는 이웃 depth에 대해서 좀더 가중치를 둔다.
- 수평, 수직방향 표준편차와 엣지 방향이 사용된다.
- 엣지의 방향은 atan2(g_y, g_x)로 계산된다.(해당 엣지 지점에서의 g_x, g_y는 공간상의 gradient이다.)
- 사실상 표준편차를 같은 비율 설정하면, 회전 자체는 의미가 없다. 아래의 가중치 사진 참조

![components](/public/img/Preprocessing/Depth_preprocessing_5.png)


### hole 영역 필터링
- 위에서 필터링이 적용된 non-hole 지역을 가지고서 필터링이 적용된다.
- 또다시 non-edge와 edge를 구분한다.
- non-edge 영역에 대해서는 이 논문에서 제안하는 "partial directional joint bilateral filter (PDJBF)"를 적용한다.
- edge 영역에 대해서 구멍을 채우기 위해서는 DJBF를 사용한다.
- 엣지 영역 채우는 것에 대한 성능을 높히기 위해서 먼저 PDJBF부터 사용한다.


#### edge와 non-edge 구멍 채우기를 위해서 필요한 엣지 방향 구하기
- 이 논문에서는 edge와 non-edge 영역에 대한 depth 구멍을 채울 때 고려하는 것은 4 방향(상하좌우) 중에서 어떤 방향에 물체의 경계(edge)가 존재하는 지를 아는 것이다.
- 이 경계가 존재하는 방향을 파악하는 실제 방법은 depth 구멍주변에서 어느쪽에 가장 가까운 color 엣지(물체의 경계)가 있는지를 찾는 것이다.
- 물체의 경계에 있는 depth 구멍에 대해서 실제로 물체에 속하는 구멍인지 아니면 경계밖에 배경에 속하는 영역의 구멍인지를 파악할 수 있어야한다.

![Alt text](/public/img/Preprocessing/Depth_preprocessing_6.png)

- 위 그림에서 맨 왼쪽의 경우, 엣지의 왼쪽 영역에 구멍이 명확하게 존재한다. 이 경우 왼쪽에서 오른쪽 방향으로의 hole filling이 적용된다. 좀더 정확히는 각 구멍마다 가장 가까운 엣지와의 거리를 4방향으로 구하고, 가장 작은 거리의 방향이 결정된다.

### Non edge 영역 구멍 채우기

- 방향이 결정되면, 우선 non-edge를 채우기 위해서, 제안하는 PDJBF를 사용한다. 수식은 아래와 같이 커널에 사용되는 이웃들을 제외하고는 기존 DJBF와 같다.

![Alt text](/public/img/Preprocessing/Depth_preprocessing_7.png)

- 여기서 필터의 사이즈는 hole filling의 성능을 개선하기 위해서 적응형으로 바뀐다.
- 특히 윈도우의 사이즈는 같은 영역에 속하는 구멍의 크기에 따라서 달라진다. 이 사이즈를 정하기 위해서 우리는 아까구한 가장 작은 거리가 아닌 다른 방향의 거리 3개중에서 작은 것을 윈도우 사이즈의 절반값으로 사용한다. 물론 max 사이즈가 있다.

![Alt text](/public/img/Preprocessing/Depth_preprocessing_8.png)

- 또한 가장 가까웟던 엣지 픽셀의 엣지 방향에 따라서 이웃 픽셀들도 다르게 사용된다. 아래의 그림과 같이 엣지가 위치한 곳의 반대 쪽에 위치한 이웃 픽셀(solid line으로 채워짐)이 사용되었다.

![Alt text](/public/img/Preprocessing/Depth_preprocessing_9.png)


- 제안한 PDJBF는 몇몇 물체의 depth 비연속성으로 구성된 영역에서 특히 효과적이었다.

### edge 영역 구멍채우기

- none-edge 영역의 구멍 채우기 이후에, 엣지 영역내의 구멍은 DJBF를 통해서 채워진다. 이때 윈도우 사이즈는 non-hole에서 사용되었던 DJBF와 달리 non-edge 영역에서 사용되었던 사이즈 방식을 사용한다.
- 이 방식은 엣지 방향을 이용하기 때문에, depth에서의 선명함을 가져갈수 있었다.


### 아직도 남은 구멍채우기
- 이후에 아직도 채워지지 않은 작은 구멍에 대해서는 간단하게 JBF로 채웠다.

## 실험에 사용된 파라미터

- DJBF에서 사용된 표준편차
  - s 표준편차 : 3
  - r 표준편차 : 0.1
  - x 표준편차 : 3
  - y 표준편차 : 1
- adaptive window size의 max값 : 11 (실행 시간과 필터링 성능에서 적절함)
- 낮은 이웃 픽셀수의 임계값 (T_n) : 10 (이정도면 작은 엣지들은 없어진다.)

### 실험결과
- directional gaussian kernel을 써서, 좀더 depth의 경계가 깔끔하게 처리되었다. 엣지 근처에 artifact들이 적어졌다.

![Alt text](/public/img/Preprocessing/Depth_preprocessing_10.png)

- 필터링된 depth를 사용해서 3차원 점으로 표현해보았다. 제안방법을 사용하였을 때 높은 성능을 보여주었다.

![Alt text](/public/img/Preprocessing/Depth_preprocessing_11.png)
