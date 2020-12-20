---
title: Tensorflow2
author: Jiny
date: 2020-10-20 19:30:08 +0800
categories: [DL, Basics]
tags: [dl, basics]
toc: false
---

# **Tensorflow2**
---

## 개발 배경

> Tensorflow1의 계산 그래프(세션) 사용의 복잡성
  - 속도 직관성 면에서 모두 떨어짐
  - Keras가 사용성을 높여주나, 보다 유기적 결합 필요

## Ecosystem

![Image](https://i0.wp.com/rubikscode.net/wp-content/uploads/2019/04/TFarch-2.jpg?w=672&ssl=1)

## 변화와 특징

- API 간소화 및 정리
  - 사라진 주요 API
    - Session
    - control_dependencies, global_variables_initializer
  - 1.x 버전 호환 API도 tf.compat.v1.Session()
- 즉시 실행 모드(eager execution): 기본 모드
  - 그래프 생성 없이 텐서 값 계산 가능 -> 연산 값 바로 확인 가능
  - Numpy 와 유사
- Session 대신 tf.function 데코레이터
- tf.keras
  - 고수준 API 문법을 케라스 기반의 tf.keras로 통일
- TPU 지원
  - 머신러닝과 딥러닝을 윟나 하드웨어

## tf.keras

- 주요 하위 모듈
  - dataset: 데이터 셋
  - models: 모델 생성 및 모델 관련 API
  - metrics: 평가 측정 관련 API
  - losses: 손실 관련 API
  - callback: 콜백 처리
  - 기타 Keras 모듈: estimator, optimizer, layers, ...
- 주요 클래스
  - Model: 계층을 한 객체로 그룹화 한 것
  - Sequential: 계층의 선형 스택
- 주요 함수
  - Input(): Keras 텐서를 인스턴스화

## 예제 코드
```python
mnist = tf.keras.datasets.mnist

(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

# 채널 차원을 추가합니다.
x_train = x_train[..., tf.newaxis]
x_test = x_test[..., tf.newaxis]

train_ds = tf.data.Dataset.from_tensor_slices(
    (x_train, y_train)).shuffle(10000).batch(32)
test_ds = tf.data.Dataset.from_tensor_slices((x_test, y_test)).batch(32)

class MyModel(Model):
  def __init__(self):
    super(MyModel, self).__init__()
    self.conv1 = Conv2D(32, 3, activation='relu')
    self.flatten = Flatten()
    self.d1 = Dense(128, activation='relu')
    self.d2 = Dense(10, activation='softmax')

  def call(self, x):
    x = self.conv1(x)
    x = self.flatten(x)
    x = self.d1(x)
    return self.d2(x)

model = MyModel()

loss_object = tf.keras.losses.SparseCategoricalCrossentropy()

optimizer = tf.keras.optimizers.Adam()

train_loss = tf.keras.metrics.Mean(name='train_loss')
train_accuracy = tf.keras.metrics.SparseCategoricalAccuracy(name='train_accuracy')

test_loss = tf.keras.metrics.Mean(name='test_loss')
test_accuracy = tf.keras.metrics.SparseCategoricalAccuracy(name='test_accuracy')

@tf.function
def train_step(images, labels):
  with tf.GradientTape() as tape:
    predictions = model(images)
    loss = loss_object(labels, predictions)
  gradients = tape.gradient(loss, model.trainable_variables)
  optimizer.apply_gradients(zip(gradients, model.trainable_variables))

  train_loss(loss)
  train_accuracy(labels, predictions)

@tf.function
def test_step(images, labels):
  predictions = model(images)
  t_loss = loss_object(labels, predictions)

  test_loss(t_loss)
  test_accuracy(labels, predictions)

EPOCHS = 5

for epoch in range(EPOCHS):
  for images, labels in train_ds:
    train_step(images, labels)

  for test_images, test_labels in test_ds:
    test_step(test_images, test_labels)

  template = '에포크: {}, 손실: {}, 정확도: {}, 테스트 손실: {}, 테스트 정확도: {}'
  print (template.format(epoch+1,
                         train_loss.result(),
                         train_accuracy.result()*100,
                         test_loss.result(),
                         test_accuracy.result()*100))
```
