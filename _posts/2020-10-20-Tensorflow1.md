---
title: Tensorflow1
author: Jiny
date: 2020-10-20 18:10:08 +0800
categories: [DL, Basics]
tags: [dl, basics]
toc: false
---

# **Tensorflow Basics**
---
## Tensor
> 벡터 계산을 단순화하기 위해 여러 같은 성질의 벡터를 한 행렬 안에 표기하고 그것을 단순화하여 표기한 것

![Tesnor](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqS1NF%2FbtqubStze09%2F5sbnC8DVd3aQsUULgjv6Kk%2Fimg.png)

- Rank
  > 텐서의 Rank는 간단히 말해서 '몇 차원 배열이냐'를 의미한다.

## Tensorflow
> Tensor를 Flow 시키면서 데이터를 처리하는 라이브러리

- 텐서는 계산 그래프 구조(Computational Graph)를 통해 노드에서 노드로 flow
  - 계산 그래프(Computation Graph)
    - 계산의 기본단위로 데이터의 흐름을 그래프로 나타냄
    - 계산 그래프 내의 텐서를 생성 변환 및 처리하는 연산을 순서대로 수행
  - 과정
    - 계산그래프의 생성 -> 실행

## Tensorflow1
- Simple XOR

```python
import tensorflow as tf

print(tf.__version__)

X = tf.placeholder(dtype=tf.float32, shape=(4, 2))
Y = tf.placeholder(dtype=tf.float32, shape=(4, 1))

INPUT_XOR = [[0,0],[0,1],[1,0],[1,1]]
OUTPUT_XOR = [[0],[1],[1],[0]]

learning_rate = 0.01
epochs = 10000

# Hidden Layer
with tf.variable_scope('hidden'):
    h_w = tf.Variable(tf.truncated_normal([2, 2]), name='weights')
    h_b = tf.Variable(tf.truncated_normal([4, 2]), name='biases')
    h = tf.nn.relu(tf.matmul(X, h_w) + h_b)

# Output Layer
with tf.variable_scope('output'):
    o_w = tf.Variable(tf.truncated_normal([2, 1]), name='weights')
    o_b = tf.Variable(tf.truncated_normal([4, 1]), name='biases')
    Y_estimation = tf.nn.sigmoid(tf.matmul(h, o_w) + o_b)

# Loss function
with tf.variable_scope('cost'):
    cost = tf.reduce_mean(tf.squared_difference(Y_estimation, Y))

# Training Layer
with tf.variable_scope('train'):
    train = tf.train.AdamOptimizer(learning_rate).minimize(cost)

# Tensorflow Session
with tf.Session() as session:

    # init session variables
    session.run(tf.global_variables_initializer())

    # log count
    log_count_frac = epochs/10

    for epoch in range(epochs):

        # Training network
        session.run(train, feed_dict={X: INPUT_XOR, Y:OUTPUT_XOR})

        # log training parameters
        if epoch % log_count_frac == 0:
            cost_results = session.run(cost, feed_dict={X: INPUT_XOR, Y:OUTPUT_XOR})
            print(cost_results)

    print("Training Completed !")
    Y_test = session.run(Y_estimation, feed_dict={X:INPUT_XOR})
    print(Y_test)
```
 - 변수: tf.Variable (초기화 필요: Weight. bias)
 - placeholder: 값 입력 없이 계산 그래프 생성 (X, Y 등 입력값 출력값)
 - session: 2.x 에서는 삭제 계산 그래프를 담고 있음