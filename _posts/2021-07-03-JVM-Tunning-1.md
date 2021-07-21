---
title: JVM Tunning 1
author: Jiny
date: 2021-07-03 14:30:00 +0800
categories: [Java, JVM]
tags: [java, java, troubleshooting]
toc: false
---
 
# JVM Tunning 1

___

## 💿 **GC**

> GC(Garbage Collection): 더이상 사용하지 않는 객체 등을 메모리에서 해제(삭제)하는 JVM의 작업

### **JVM**

![image](https://www.journaldev.com/wp-content/uploads/2014/05/Java-Memory-Model.png)
_JVM Heap 메모리 구조_

### **Java에서 GC를 도입이 가능했던 이유**

> weak generational hypothesis 가설로 인해 GC 도입이 가능해짐
> - 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 됨
> -  오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재

JVM 에서는 오래된 객체를 구별하기 위해 메모리를 여러 영역으로 나눔