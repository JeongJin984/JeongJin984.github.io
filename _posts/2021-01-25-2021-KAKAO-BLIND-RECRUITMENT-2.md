---
title: Spring JPA
author: Jiny
date: 2021-01-24 17:16:00 +0800
categories: [Java, Algo]
tags: [kako, test, coding]
toc: false
---

# 2021 KAKAO BLIND RECRUITMENT-2
___

## 문제 설명
레스토랑을 운영하던 스카피는 코로나19로 인한 불경기를 극복하고자 메뉴를 새로 구성하려고 고민하고 있습니다.
기존에는 단품으로만 제공하던 메뉴를 조합해서 코스요리 형태로 재구성해서 새로운 메뉴를 제공하기로 결정했습니다. 어떤 단품메뉴들을 조합해서 코스요리 메뉴로 구성하면 좋을 지 고민하던 스카피는 이전에 각 손님들이 주문할 때 가장 많이 함께 주문한 단품메뉴들을 코스요리 메뉴로 구성하기로 했습니다.
단, 코스요리 메뉴는 최소 2가지 이상의 단품메뉴로 구성하려고 합니다. 또한, 최소 2명 이상의 손님으로부터 주문된 단품메뉴 조합에 대해서만 코스요리 메뉴 후보에 포함하기로 했습니다.

예를 들어, 손님 6명이 주문한 단품메뉴들의 조합이 다음과 같다면,
(각 손님은 단품메뉴를 2개 이상 주문해야 하며, 각 단품메뉴는 A ~ Z의 알파벳 대문자로 표기합니다.)

|손님 번호|주문한 단품메뉴 조합|
|---|---|
|1번 손님|A, B, C, F, G|
|2번 손님|A, C|
|3번 손님|C, D, E|
|4번 손님|A, C, D, E|
|5번 손님|B, C, F, G|
|6번 손님|A, C, D, E, H|


가장 많이 함께 주문된 단품메뉴 조합에 따라 스카피가 만들게 될 코스요리 메뉴 구성 후보는 다음과 같습니다.


|코스종류|메뉴 구성|설명|
|---|---|---|
요리 2개 코스|A, C|1번, 2번, 4번, 6번 손님으로부터 총 4번 주문됐습니다.|
요리 3개 코스|C, D, E|3번, 4번, 6번 손님으로부터 총 3번 주문됐습니다.|
요리 4개 코스|B, C, F, G|1번, 5번 손님으로부터 총 2번 주문됐습니다.|
요리 4개 코스|A, C, D, E|4번, 6번 손님으로부터 총 2번 주문됐습니다.|


## 문제

각 손님들이 주문한 단품메뉴들이 문자열 형식으로 담긴 배열 orders, 스카피가 **추가하고 싶어하는** 코스요리를 구성하는 단품메뉴들의 갯수가 담긴 배열 course가 매개변수로 주어질 때, 스카피가 새로 추가하게 될 코스요리의 메뉴 구성을 문자열 형태로 배열에 담아 return 하도록 solution 함수를 완성해 주세요.

## 제한 사항

- orders 배열의 크기는 2 이상 20 이하입니다.
orders 배열의 각 원소는 크기가 2 이상 10 이하인 문자열입니다.
- 각 문자열은 알파벳 대문자로만 이루어져 있습니다.
- 각 문자열에는 같은 알파벳이 중복해서 들어있지 않습니다.
- course 배열의 크기는 1 이상 10 이하입니다.
- course 배열의 각 원소는 2 이상 10 이하인 자연수가 오름차순으로 정렬되어 있습니다.
- course 배열에는 같은 값이 중복해서 들어있지 않습니다.
- 정답은 각 코스요리 메뉴의 구성을 문자열 형식으로 배열에 담아 사전 순으로 오름차순 정렬해서 return 해주세요.
- 배열의 각 원소에 저장된 문자열 또한 알파벳 오름차순으로 정렬되어야 합니다.
- 만약 가장 많이 함께 주문된 메뉴 구성이 여러 개라면, 모두 배열에 담아 return 하면 됩니다.
- orders와 course 매개변수는 return 하는 배열의 길이가 1 이상이 되도록 주어집니다.



|ORDERS|COURSE|RESULT|
|------|---|----|
|["ABCFG", "AC", "CDE",<br/> "ACDE", "BCFG", "ACDEH"]|[2,3,4]|["AC", "ACDE", <br/>"BCFG", "CDE"]|
|["ABCDE", "AB", "CD",<br/> "ADE", "XYZ", "XYZ",<br/> "ACD"]|[2,3,5]|["ACD", "AD", <br/>"ADE", "CD", <br/>"XYZ"]|
|["XYZ", "XWY", "WXA"]|[2,3,4]|["WX", "XY"]|

## 풀이

1. Brutal Force Search
   
> 기준 수열의 모든 부분 수열을 찾고 그 수열의 모든 원소가 비교 수열에 있으면(메뉴가 겹치면) map을 증가

```java
import java.util.*;

class Solution {
    
    String[] orders;
    int[] course;
    
    HashMap<String, Integer> map = new HashMap<>();
    int max = 0;
    
    
    void DFS(String s, int i, String order, int length) {

        if(s.length() == length) {
            boolean contains = true;
            for(String s2 : orders) {
                contains = true;
                for(char c : s.toCharArray()) {
                    if(!s2.contains(c+"")) contains = false;
                }
                if(contains) {
                    char[] t = s.toCharArray();
                    Arrays.sort(t);
                    s = new String(t);
                    if(map.containsKey(s)) map.put(s, map.get(s)+1);
                    else map.put(s, 1);
                }
            }
            if(max < map.get(s)) max = map.get(s);
        }

        if(s.length() < length && i < order.length()) {
            DFS(s+order.charAt(i), i+1, order, length );
            DFS(s, i+1, order, length);
        }
    }
    
    public String[] solution(String[] orders, int[] course) {
        this.orders = orders;
        this.course = course;
        
        List<String> answerList = new ArrayList<>();

        for (int i : course) {
            map = new HashMap<>();
            max = 0;
            for (String order : orders) {
                DFS("", 0, order, i);
            }
            System.out.println();
            for (String s : map.keySet()) {
                int v = map.get(s);
                if(v == max && max != 1) answerList.add(s);
            }
        }

        Collections.sort(answerList);
        String[] answer = new String[answerList.size()];

        for (int i=0; i< answer.length; i++){
            answer[i] = answerList.get(i);
        }
        
        return answer;
    }
}
```

1. Refactor(모든 부분 수열을 구한 후 가장 많이 겹친)

> 가능한 모든 부분 수열을 모두 구한 후 가장 많이 겹친 키 값을 메뉴로 정한다.
```java
import java.util.*;

class Solution {
    HashMap<String, Integer> map;
    char[] subCharArray;
    int max = 0;
    List<String> maxString = new ArrayList<>();

    void combination(char[] order, int subIndex ,int index, int length) {
        if(subIndex == length) {
            String subString = String.valueOf(subCharArray);
            int temp = map.getOrDefault(subString, 0);
            int value = temp + 1;
            map.put(subString, value);
            if(max <= value) {
                max = value;
            }
            return;
        }

        for(int i = index; i<order.length; i++) {
            subCharArray[subIndex] = order[i];
            combination(order, subIndex+1, i+1, length);
        }
    }
    
    public String[] solution(String[] orders, int[] course) {
        String[] answer = {};
        for (int l : course) {
            map = new HashMap<>();
            max = 0;
            for (String orderString : orders) {
                subCharArray = new char[l];
                char[] orderArray = orderString.toCharArray();
                Arrays.sort(orderArray);
                combination(orderArray, 0, 0, l);
            }
            for (String s : map.keySet()) {
                if(map.get(s) == max && max != 1) maxString.add(s);
            }
        }
        Collections.sort(maxString);
        answer = new String[maxString.size()];
        return maxString.toArray(answer);
    }
}
```