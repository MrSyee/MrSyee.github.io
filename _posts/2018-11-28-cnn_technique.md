---
title: CNN의 성능, 효율을 높이기 위한 기법들
layout: post
categories: [Image Processing]
tags: [ai, image, cnn]
use_math : true
---
## Introduction
CNN(Convolution Neural Network)은 주로 이미지 처리에 쓰이는 인공신경망 입니다. 	CNN이 처음 제안된 후부터 대부분의 연구들은 CNN의 성능에 주목하는 연구들이 많았습니다. 그러나 최근에는 Mobile 환경 또는 임베디드 시스템과 같은 작은 환경들에 CNN을 적용하기 위해 모델을 경량화하는 연구가 많이 진행되고 있습니다. 이러한 CNN의 성능 향상 또는 경량화하는 기법들을 간단하게 정리해서 소개해보려합니다.

#### Notation
- M : 입력 채널 수
- X, Y : 입력 사이즈 (width, height)
- N : 필터 수
- K : 필터 커널 사이즈
- P : 패딩 사이즈

#### Convolution layer에 의한 Size 계산
- $$ {X(or Y) - K + 2P \over stride} + 1 $$

### 3 x 3 Convolutional filters
- 3x3 필터를 여러개 쌓는 것이 그 이상의 필터를 한 층 사용하는 것과 비교했을 때 성능은 유지되면서 파라미터 수는 적어진다.
- 파라미터 수 : N(MKK +1)
- if M=3, N =32
- 5x5 layer 1개 : N(25M + 1) = 25MN +N = 2432
- 3x3 layer 2개 : N(9M + 1) + N(9M +1) = 18MN + 2N = 1792
- 큰 size의 필터를 사용했을 때보다 출력되는 feature size는 같으면서 필요한 파라미터 수가 줄어든다.
- ref : VGGNet

### Residual Learning (Skip Connection)
- 기존의 네트워크에 일종의 지름길(Skip Connection)을 추가한 구조
- weight layer를 거친 output과 input을 더해주는 형태로 구성. 이를 **Residual learning block** 이라 함.
- weight layer는 input과의 **차이** 만 학습하면 되기 때문에 학습이 더 좋아짐.
- Skip Connection을 추가하면 성능이 올라가기 때문에 기존보다 layer수를 줄여도 성능이 유지된다.
- ref : ResNet

### Point-wise Convolutional
- 1 x 1 Conv filter로 channel 단위의 convoution 연산을 한다. (1 x 1 x N)
- channel간의 불필요한 뉴런을 없애는 pruning 효과.
- channel reduction.
- 파라미터 수 : N(M + 1)

### Depth-wise Convolution
- 필터 수가 채널 수가 같은 conv filter.
- 각 채널에 대한 feature를 추출할 수 있다.
- 파라미터 수 : N(MKK + 1)

### Depth-wise Separable Convolution
- Depth-wise conv + Point-wise conv 순서로 구성.
- 채널 간의 conv 연산과 각 채널에 대한 conv연산을 분리해서 처리할 수 있는 구조.
- 파라미터 수 : M(KK + 1) + N(M + 1)
- 3x3 필터 기준으로 기존 모델과 **약 9배** 의 모델 사이즈(파라미터 수) 차이를 보여줌.
- ref : Xception, Mobilenet v1

### SqueezeNet
