---
title: JVM Tunning 2
author: Jiny
date: 2021-07-03 16:30:00 +0800
categories: [Java, JVM]
tags: [java, java, troubleshooting]
toc: false
---
 
# JVM Tunning 2

___

## 💿 **IBM JVM**

### **optthruput GC**

- **옵션** 
  - \-Xgcpolicy: optthuput
- **설명**
  - default GC 알고리즘.
  - mark-sweep-compact 단계를 수행하여 STW로 일시정지됨.
  - application이 복잡해지고 그에 따라 heap이 커지면 GC 수행에 따른 멈춤시간도 증가
- **장점**
  - Throughput이 향상
- **단점**
  - STW로 인한 Response Time 감소

### **optavgpause GC**

- **옵션**
  - \-Xgcpolicy: optavgpause
- 설명
  - opthuput의 담점인 STW 시간을 보완하고자 함
  - GC를 위한 Thread를 추가적으로 만들어서 concurrent mark를 수행하게 하여 실제 GC가 발생할 때 전체 heap 영역에 대한 mark 작업을 수행할 필요없이 sweep 단계를 수행
- 장점
  - 전반적인 response time이 향상됨
- 단점
  - Throughput(처리량)이 감소

> 위의 둘을 확인하였을 때, Throughput(처리량)과 response time은 반비례관계에 있다는 것을 알 수 있음

### **gencon GC**

- **옵션**
  - \-Xgcpolicy:gencon
- 설명
  - sun JVM과 동일한 구조
  - response time 과 throughput을 모두 향상시킬 수 있음
  - Heap을 Nursery Space 와 Tenured Space로 구성하며 Nursery Space 내에 다시 Allocate Space와 Survivor Space로 나눠진다.
  - Nursery Space의 gc는 minor gc로, Copy 방식(Alive Object를 Allocate Space에서 Survivor Space로 복사하는 것)
  - Tenured space의 gc는 Full gc로 Mark+Sweep+Commpaction이 수행됨
- 장점
  - response time과 throughput이 동시에 향상됨. 단 튜닝이 필요할 수 있음
- 단점
  - CPU 오버헤드가 발생

### **subpool GC**

- **옵션**
  - \-Xgcpolicy: subpool
- **설명**
  - Java Heap을 여러개의 sub poll로 나누어 관리하는 기법을 사용
  - 16 CPU 이상의 서버 환경에서만 사용할 것을 권장
  - Sum JVM의 Parallel GC와 동일한 방식

___

## 💿 **Sun HotSpot JVM**

### **Serial GC**

- **옵션**
  - \-XX:+UseSerialGC
- **설명**
  - Serial GC는 적은 메모리와 CPU 코어 개수가 적을 때 적합한 방식
  - 단일 Thread가 GC를 수행

### **Parallel GC**

- **옵션**
  - \-XX: +UseParallelGC
- **설명**
  - Parallel GC는 Serial GC와 기본적인 알고리즘은 같음
  - Serial GC는 처리하는 Thread가 하나인 것에 비해 Parallel GC는 GC를 처리하는 Thread가 여러개임
  - 따라서 빠름 그러나 메모리가 충분하고 코어 개수가 많을 때 사용해야 한다.

### **Parallel Old GC**

- **옵션**
  - \-XX: +UseParallelOldGC
- **설명**
  - Parallel과 비교하여 Old 영역의 GC 알고리즘만 다르다.
  - Mark-Summary-Compaction 단계를 거침
    - 앞서 GC를 수행한 영역에 대해서 별도로 살아있는 객체를 식별한다는 점에서 Mark-Sweep-Compaction 알고리즘의 Sweep 단계와 다름

### **CMS GC**

- **옵션**
  - \-XX: UseConcMarkSweepGC
- **설명**
  - initial Mark > Concurrent Mark > Remark > Concurrent Sweep 방식으로 진행됨
  - Concurrent Mark, Concurrent Sweep의 경우 다른 Thread가 실행중인 상태에서 동시에 진행됨
    - STW 시간이 짧음
  - 모든 Application의 response time이 중요할 때 사용하며, 대신 메모리와 CPU 사용량 높음
  - 기본적으로 Compaction 작업이 제공되지 않음

### **G1GC**

- **옵션**
  - \-XX: +UseG1GC
- **설명**
  - 기존 JVM 메모리 구조와는 다르게, Heap 영역이 1MB ~ 32MB이내의 고정된 크기로 2000여개 영역(Region)으로 분할되어 있음
  - 각 Region은 기존 JVM heap 영역이었던 New, Survivor, Old, Perm 영역을 각각 담당하지만 동적으로 역할이 변경될 수 있음
  - Region 안에 있는 객체들이 다른 Region에 있는 객체들에 의해서 참조되는지를 관리하고 Region 단위로 Live Object가 있는지 없는지를 판단하여 해당 Region이 다차면 살아있는 object들을 다른 Region으로 copy 한 후, 해당 Region을 비움

___

## 💿 **JVM Tunning Option**

### **IBM JVM**

- \-Xms
  - JVM 시작시 heap size
- \-Xmx
  -  최대 heap size
- \-Xmn, \-Xmns, \-Xmnx
  - New 영역의 크기(최소=최대, 최소, 최대)
- \-Xmo, \-Xmos, \-Xmox
  - Old 영역의 크기(최소=최대, 최소, 최대)
- \-Xdisableexplicitgc
  - System.gc()에 의한 GC를 비활성화
- \-Xgcthreads
  - Parallel GC 작업을 할 Thread 개수를 지정.
  - 만일 여러 Process가 CPU를 나누어 쓰는 환경이라면 이 값을 낮출 필요가 있다.
- \-Xloa
  - LOA(Large Object Area)를 사용할 지의 여부를 결정
  - Default는 활성화 상태
  - large object area는 Heap Size의 5%를 차지하는데 AP의 특성상 비교적 큰 객체(5MB)이상의 생성이 빈번한 경우에는 이 영역을 늘려줄 필요가 있다.
- \-Xloainitial, \-Xloamaximum, \-Xloaminimum
  - LOA의 초기 크기, 최대크기, 최소 크기를 지정한다. 0~1 사이의 값을 지정한다.
- \-Xmaxe, \-Xmine
  - Heap Expansion이 증가할 최대/최소 크기를 지정한다.
- \-Xmaxf, \-Xminf
  - Heap 크기를 조정할 Free Memory의 크기를 결정한다. Default 값은 60%, 30%이다.
    - Free Memory가 전체 Memory의 60% 이상이 되면 Heap Shrinkage가 발생하고
    - 30% 이하면 Heap Expansion이 발생

### **Sun HotSpot JVM**

- \-Xms
  - JVM 시작 시 Heap Size
- \-Xms
  - 최대 Heap Size
- \-XX: New Ratio
  - New 영역과 Old 영역의 비율
- \-XX: New Size
  - New 영역의 크기
- \-XX: MaxNewSize
  - New generation의 heap size 최대값을 설정
- \-XX: SurvivorRatio
  - Eden 영역과 Survivor 영역의 비율
- \-Xcompactgc
  - 모든 garbage collections(system 또는 global)에 대하여 압축을 가능하게 한다.
- \-XnoCompactgc
  - 모든 garbage collections(system 또는 global)에 대하여 압축을 불가능하게 한다.
- \-Xcompactexplicitgc
  - System.gc()가 호출 될 때마다 모든 GC 시스템에서 압축을 가능하게 한다.
- \-Xnocompactexplicitgc
  - System.gc()가 호출에 의한 압축을 불가능하게 함.
  - 압축은 -Xcompactgc를 명시하거나 또는 compaction triggers를 만나면 global garbage collections에서 발생
- \-Xnoclassgc
  - Class garbage collection을 비활성화 한다.
  - JVM에 의해서 더이상 사용되지 않는 Java Class들과 연관된 storage의 garbage collection을 끈다.
  - 즉 dynamic 클래스 로드 해제를 사용 불가능하게 한다.
