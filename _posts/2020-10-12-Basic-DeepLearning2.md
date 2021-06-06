---
title: Deep Learning Basics2
author: Jiny
date: 2020-10-12 18:13:08 +0800
categories: [DL, Basics]
tags: [dl, basics]
toc: false
---

## 용어 정리
###epoch
- One Epoch is when an ENTIRE dataset is passed forward and backward through the neural network only ONCE

###batch
- Total number of training examples present in a single batch.

### mini batch 경사하강
- 전체 데이터를 batch_size개씩 나눠 배치로 학습(배치 크기를 사용자가 지정)

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbkVbjU%2FbtqEtOUJD9H%2FL9KHdOnSukjHnhlRwRERy1%2Fimg.png)
- BGD보다 계산량이 적다. (Batch Size에 따라 계산량 조절 가능)
- Shooting이 적당히 발생한다. (Local Minima를 어느정도 회피할 수 있다.)

#### @ 학습 절차
1. 훈련 데이터의 일부를 무작위로 가져옴
2. 기울기(gradient) 산출
3. 가중치요소 들을 기울기 방향으로 갱신
4. 반복

#### @ 시험 추가
> validation set의 추가로 정확한 성능을 체크 할 수 있다.


### 성능 개선
1. 오류 역전파
  >asdf

![image](https://lh3.googleusercontent.com/proxy/WQXtg6S8aXyADdcIKIuM2qO6SywQGFoKCVxWjBIcrLSdFbtvKaXtq20bvpt-yUTvQAPP2eVgi1JM74tbw3Khy1MI3PzmgQONFprzaommxH_5GCEuxUGLugeXAUQ-NJxIim4TQMX-clHvRNCSf1a19NLyKAsQ7AhkcfT6A8pIBI4NXg8kNhvUdm0NZV5BDRRarGtTSQP3Te6c5I34IqA4QSlQetg-4b05pNKqIYYX7oTZnVxda-tQ1GnT3Q989UD63YJ5qNEBzPDePDxYeBmk1c89ozkbZ65eSBm35n8hlv4)

미분값을 통해 가중치를 개선하여 fitting 시키는 방법

