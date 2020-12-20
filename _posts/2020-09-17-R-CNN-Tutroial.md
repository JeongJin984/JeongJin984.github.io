---
title: R-CNNs Tutorial
author: Jiny
date: 2020-09-17 18:12:55 +0800
categories: [Vision, CNN]
tags: [rcnn, dl]
toc: false
---

# Intorduction
기존의 Object detection은 입력영상안의 0~N개의 모든 클래스에 대해 Classifiaction을 수행해야 합니다. 
classification 과 localization을 수행하는 법에 대해서 알아보겠습니다.

(참고: 이 포스트는 <https://blog.lunit.io/2017/06/01/r-cnns-tutorial/>를 참고하여 만들어 졌음을 알립니다.)

## Native Approach
처음 떠오르는 방법은 가능한 모든 영역에 대해 sliding window 방식으로 이미지를 탐색하면 classification을 수행하는 것입니다.
하지만 이 방법은 시간이 너무 오래 걸리죠

## Region Proposals
![Desktop View](https://bloglunit.files.wordpress.com/2017/05/20170525-research-seminar-google-slides-2017-05-25-16-45-08.png?w=661&h=173)
> 입력 영상에서 물체가 있을 법한 영역을 빠른 속도로 찾아내는 알고리즘

- selective search
- Edge boxes 

위 2개의 알고리즘이 보편적으로 좋은 성능과 속도를 보여 줍니다.

## R-CNN
> R-CNN = Region Proposal + CNN

1. 입력 이미지
2. Region Proposals
3. cropping(영역을 이미지로부터 잘라냄) 이미지를 wrapping(동일한 크기로)한 후 CNN으로 Classification -> feature 추출
4. Region Proposal에 대한 classification

위의 과정을 통해 높은 성능을 낼 수 있지만 localization에 성능이 취약합니다. 즉 정확한 위치를 알아내기가 쉽지 않습니다.
이는 CNN이 물체가 중앙에 있지 않아도 Classification에 좋은 성능을 나타내기 때문이라고 하는데요.
이를 보안하기 위해 bounding-box refressor을 논문에서 제안하고 있습니다. 이 모델의 목표는 region proposal(P)을 정답위치(G)로 mapping할 변환을 학습하는 것입니다.
하지만 R-CNN은 region Proposal 개수 만큼의 CNN 연산이 필요하기 때문에 속도가 느리고 여러 stage를 통해 학습이 진행 되는 등의 단점이 있습니다.

## Fast R-CNN
![Desktop View](https://bloglunit.files.wordpress.com/2017/05/20170525-research-seminar-google-slides-2017-05-26-17-14-03.png?w=666&h=282)

R-CNN의 가장 큰 단범은 region마다 이미지를 cropping 한 뒤 CNN 연산을 한다는 것인데요
이러한 점을 개선하기 위해 cropping 과정을 image 레벨이 아닌 feature map level에서 수행하면 CNN 연산을 1번으로 줄일 수 있다는 장점이 있습니다.

하지만 위의 과정을 위해서는 feature 크기가 각 example 마다 동일해야 합니다. wrapping을 할 수도 있지만 그 과정 속에서 생기는 손실이 효율을 떨어뜨립니다.
따라서 feature map ㅡ의 다양한 크기의 region으로부터 일정한 길이의 feature을 추출해 닐 방법이 필요합니다.

## Spatial Ptramid Pooling, ROI Pooling
다양한 크기의 입력으로 부터 일정한 크기의 feature을 추출하기 위해서 Bad of words(Bow)라는 알고리즘을 사용하였지만 위치 정보를 모두 잃어버린다는 단점이 존재했습니다.

> 그렇다면 이미지를 여러개의 일정 개수의 지역으로 나눈뒤 Bow를 적용하여 지역적인 정보를 어느정도 유지할 수 있습니다.

![Desktop View](https://bloglunit.files.wordpress.com/2017/05/1406-4729-pdf-2017-05-26-20-11-59.png)

SPPNet[^footnote]은 SPP 기법을 CNN feature map에 적용할 수 있음을 보여 준 논문입니다.
SPP Layer는 feature map 상의 특정 영역에 대해 고정된 개수의 bin 영역에 대해 pooling 을 함으로서 고정된 길이의 feature vector를 가져 올 수 있습니다.
Fast R-CNN에서는 이러한 SPP layer의 single level pyramid 만을 사용하여 이를  RoI Pooling layer라 명창하였습니다.
![Desktop View](https://bloglunit.files.wordpress.com/2017/05/20170525-research-seminar-google-slides-2017-05-29-19-06-26.png?w=768&h=180)

## R-CNN vs Fast R-CNN

![Desktop View](https://bloglunit.files.wordpress.com/2017/05/20170525-research-seminar-google-slides-2017-05-30-19-42-30.png?w=663&h=379)

- R-CNN
  1. ImageNet classification 데이터로 ConvNet을 pre-train 시켜 모델 M을 얻습니다.
  2. M을 기반으로, object detection 데이터로 ConvNet을 fine-tune 시킨 모델 M'을 얻습니다.
  3. object detection 데이터 각각의 이미지에 존재하는 모든 region proposal들에 대해 모델 M'으로 feature vector F를 추출하여 저장합니다.
    - 추출된 F를 기반으로 classifier (SVM)을 학습합니다.
    - 추출된 F를 기반으로 linear bounding-box regressor를 학습합니다.

- Fast R-CNN
  - R-CNN에서는 Softmax classifier와 linear bounding-box regressor를 따로 학습했습니다.
    - 반면, Fast R-CNN에서는 두 함수의 loss를 더한 multi-task loss를 기반으로 동시에 두 가지 task를 학습합니다.
  - R-CNN에서는 여러 장의 이미지에서 랜덤하게 N개의 영역을 샘플링한 mini-batch를 구성하여 학습을 진행하였습니다. 이러한 방식을 Fast R-CNN에 적용할 경우, 샘플링한 N개의
	  영역이 K개의 이미지로부터 오게 되고, K가 N에 가까울 확률이 높습니다. 각 영역의 feature를 얻기 위해서는 K번의 CNN 연산이 필요합니다. (region-wise sampling)
    - Fast R-CNN에서는 이를 개선하기 위해 R-CNN의 RoI 샘플링 방식 대신, 1장 또는 2장 이내의 이미지에서 N개의 영역을 샘플링한 mini-batch를 사용함으로써 Fast R-CNN 학습에 필요한 CNN 연산량을 효율적으로 줄일 수 있었습니다. (image-centric sampling)

## Faster R-CNN
R-CNN: R-CNN에서는 3가지 모듈 (region proposal, classification, bounding box regression)을 각각 따로따로 수행합니다.
> region proposal 추출 → 각 region proposal별로 CNN 연산 → classification, bounding box regression

Fast R-CNN: R-CNN에서는 3가지 모듈 (region proposal, classification, bounding box regression)을 각각 따로따로 수행합니다.
> region proposal 추출 → 전체 image CNN 연산 → RoI projection, RoI Pooling → classification, bounding box regression

Fast R-CNN을 통해 속도 문제를 어느정도 해결하였지만 여전히 region-proposal인 selective search를 CNN 외부에서 연산하므로 ROI 생성단계가 병목이 됩니다.
> 따라서 Faster R-CNN에서는 detection에서 쓰인 conv feature을 RPN에서 공유함으로서 ROI 생성 역시 CNN 내부에서 수행하여 속도 향상을 노렸습니다. 즉 Selective search가 아닌 CNN 내부에서 Region Proposal을 한다는 것입니다.

Faster R-CNN은 Fast R-CNN 구조에서 conv featrue map 과 RoI Pooling 사이에 RoI를 생성하는 Region Proposal Network가 추가된 구조입니다.
![Desktop View](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdhq4iV%2FbtqBaAFDl4d%2FIZdxlDX5mkPMdnoKy2f2k0%2Fimg.png)

그리고 Faster R-CNN에서는 RPN 네트워크에서 사용할 CNN과 Fast R-CNN에서 classification, bbox regression을 위해 사용한 CNN 네트워크를 공유하자는 개념에서 나왔습니다.
![Desktop View](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3fKvm%2FbtqA7qcGUyK%2FRtIY6qVkJ6yerNqBUnV0h1%2Fimg.png)

위의 그림에서 볼 수 있듯이 CNN을 통과하여 생성된 conv feature map이 RPN에 의해 RoI를 생성합니다.(Original image에서의 RoI)
(그래서 코드 상에서도 anchor box의 scale은 original image 크기에 맞춰서 생성하고 이 anchor box와 network의 output 값 사이의 loss를 optimize하도록 훈련시킵니다.)
이렇게 feature map에 RoI가 투영되고 나면 FC layer에 의해 classification과 bbox regression이 수행됩니다.

>bbox regression이란 물체가 있을 법한 위치를 찾았으니 해당 물체를 classificate 했습니다. 하지만 selective search를 통해 찾은 박스의 위치가 부정확 하므로 이를 교정해 주는 것이 bbox regression입니다. 즉 CNN을 통과하여 추출된 벡터와 x, y, w, h를 조정하는 함수의 웨이트를 곱해서 바운딩 박스를 조정해주는 선형 회귀를 학습시키는 것입니다.

이러한 이유에서 입력 영상의 크기는 어떤 크기가 되어도 상관 없지만 성능상 vgg의 경우 244x224, resNet의 경우 min : 600, max : 1024 등.. 으로 맞춰줄때 성능이 가장 좋기 때문에 크기를 고정해 주는 것이 좋습니다.
즉 resize 할때 손실을 RoI pooling에서도 존재하는데 이 역시 최소화 하기 위해

### FC layer 대신 GAP(Global Average Pooling)을 사용하는 추세 
라고 합니다. GAP는 input 사이즈에 관계없이 1 value로 average pooling을 하여 filter의 개수만 고정되어 있으면 된다고 합니다.

## Reverse Footnote

[^footnote]: Spatial Pyramid Pooling in Deep Convolutional Networks for Visual Recognition, K. He et al, ECCV14..