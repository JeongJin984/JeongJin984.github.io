---
title: Golang Slice
author: Jiny
date: 2021-09-02 12:40:00 +0800
categories: [Lang, Go]
tags: [go, golang, slice]
toc: false
---

# Slice
___

## 💿 **Array**

> An array type definition specifies a length and an element type.

![image](https://go.dev/blog/slices-intro/slice-array.png)

- Go’s arrays are values.
  - it is not a pointer to the first array element (as would be the case in C).
  - This means that when you assign or pass around an array value you will make a copy of its contents.
  - To avoid the copy you could pass a pointer to the array, but then that’s a pointer to an array, not an array.

```go
func main() {
	arr := [...]int{1,2,3,4,5}
	fmt.Printf("%v, %p\n", arr, &arr) 
}
// result: [1 2 3 4 5], 0xc000122030
```

> 즉 Array의 포인터를 사요용 순 있지만 C 처럼 Array 타입은 주소값을 가지지 않고, 값을 그대로 가진다.

___

## 💿 **Slice**

> 연속적 데이터의 메타데이터(주소, len, cap)을 가지는 값타입 변수

![image](https://go.dev/blog/slices-intro/slice-1.png)

- 이것도 포인터를 가지고 있을 뿐이지 값타입 변수이다.(참조형 변수 아님)
- Array를 잘라서 Slice가 되는 개념이 아닌 Array의 메타데이타를 뽑아서 Slice타입으로 변환하는 개념
  - 미묘하지만 어감 차이인것 같음...
- 생성 함수: func make([]T, len, cap) []T

### **증명**

```go
func main() {
	arr := [...]int{1,2,3,4,5}
	slice1 := arr[:]
	slice2 := arr[:]

	fmt.Printf("%p %p \n", &slice1, &slice2)
	fmt.Printf("%p %p \n", slice1, slice2)
}
// 0xc0000ac018 0xc0000ac030 (slice1,2의 주소)
// 0xc0000b0030 0xc0000b0030 (slice1,2가 가진 배열 주소)
```
- 즉 메타데이터를 뽑아서 Slice를 생성하면 매번 새로운 Slice(다른 주소)가 생성
- 하지만 생성된 Slice가 가지는 배열은 모두 같은 놈임

```go
func main() {
	arr := [...]int{1,2,3,4,5}

	slice1 := arr[:]
	fmt.Printf("%p %p %d %d \n", &slice1, slice1, len(slice1), cap(slice1))

	slice2 := append(slice1, 6)
	fmt.Printf("%p %p %d %d \n", &slice2, slice2, len(slice2), cap(slice2))

	slice21 := append(slice2, 7)
	fmt.Printf("%p %p %d %d \n", &slice21, slice21, len(slice21), cap(slice21))

	slice3 := append(slice2, 8,9,10,11)
	fmt.Printf("%p %p %d %d \n", &slice3, slice3, len(slice3), cap(slice3))

	slice4 := append(slice3, 12,13,14,15,16,17,18,19,20,21)
	fmt.Printf("%p %p %d %d \n", &slice4, slice4, len(slice4), cap(slice4))
}
/*
 result:
 0xc000126018 0xc00012a030 5 5 
 0xc000126048 0xc000138000 6 10 
 0xc000126078 0xc000138000 7 10 
 0xc0001260a8 0xc000138000 10 10 
 0xc0001260d8 0xc00013c000 20 20 
*/
```
             
                                                         
- cap를 넘을 때 마다 새로운 주소를 가진 배열이 할당됨
- append할때마다 새로운 slice가 생성되지만 cap을 넘지 않으면 새로운 배열이 생성되지는 않음