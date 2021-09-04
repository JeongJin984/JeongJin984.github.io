---
title: Prometheus 1
author: Jiny
date: 2021-09-01 20:30:00 +0800
categories: [Metric, Prometheus]
tags: [metric, prometheus, monitoring]
toc: false
---

# Prometheus 

___

## 💿 **Node Exporter**

> 주로 운영 시스템 커널에서 오는 머신 수준의 메트릭을 게시한다.

- 즉 머신 자체를 모니터링하기 위한 것

### **CPU 수집기**

- 주요 메트릭: node_cpu_seconds_total: 각 모드에서 CPU마다 얼마나 많은 시간을 소비했는지 나타내는 카운터
  - 레이블: cpu, mode
- avg without(cpu, mode)(rate(node_cpu_seconds_total{mode="idle"}[1m])
  - CPU마다 초당 유휴 시간을 계산한 다음 머신에서 모든 CPU의 평군을 계산

### **FileSystem 수집기**

> df 명령어에서 얻을 수 있는 것처럼 마운트 된 파일 시스템의 메트릭을 수집

- --collector.filesystem.ignored-mount-points와 --collector.filesystem.ignored-fs-types는 포함되는 파일 시스템을 제한할 수 있음
 - 루트 사용자로 Exporter를 실행시키지 않아서 마운트 지점에서 statfs 시스템 호출을 사용할 수 있는 파일 접근 권한 확인
- 모든 메트릭에 node_filesystem_ 접두어가 붙어있음
- node_filesystem_avail_bytes, node_filesystem_free_bytes
 - 전자: 일반 사용자들이 이용할 수 있는 공간 
   - 사용된 디스크 공간을 계산하는 경우: node_filesystem_avail_bytes / node_filesystem_size_bytes
 - 후자: 전체 inode 개수와 이들 중 사용가능한 inode 개수 이를 통해 파일 시스템이 갖고 있는 대략적인 파일 개수를 알 수 있음(df -i)

### **diskstats 수집기**

> /proc/diskstats에서 얻은 디스크 I/O 메트릭을 게시한다.

- 기본적으로 --collector.diskstats.ignored-device는 파티션과 루프백 장치같이 실제 디스크가 아닌 경우는 제외

### **netdev 수집기**

> node_network_ 접두어와 device 레이블을 사용해 네트워크 디바이스에 대한 메트릭을 게시한다.

- 주요 메트릭
  - node_network_receive_bytes_total
  - node_network_transmit_bytes_total
  - node_network_receive_packets_total
  - node_network_transmit_packets_total

### **meminfo 수집기**

> /proc/meminfo에서 얻는 모든 메모리 관련 표준 메트릭을 갖고 있음

- 주요 메트릭
  - node_memory_MemTotal_bytes: 머신의 실제 메모리 총량
    - 하지만 사용된 메모리에 대한 메트릭은 없음
    - 다른 메트릭에서 사용되지 않은 메모리 양도 계산해야함
  - node_memory_MemFree_bytes: 사용되지 않는 메모리의 총량
    - 보유한 모든 메모리가 여유분이라는 것은 아님
    - 이론적으로, 페이지 캐시는 쓰기 버퍼처럼 다시 확보될 수 있지만 성능에 부정적 영향을 줄 수 있음
    - slab과 페이지 테이블같이 메모리를 이용하는 다양한 커널 구조들도 있음
  - node_memory_MemAvailable는 커널에서 얼마나 많은 메모리를 실제로 사용할 수 있는지에 대한 것
    - 메모리 소모를 감지하는데 사용

### **hwmon 수집기**

> 베어 메탈의 경우 hwmon 수집기는 온도, 팬 속도 등 node_hwmon_ 접두어를 갖는 메트릭을 제공

### **stat 수집기**

> /proc/stat에서 제공하는 다양한 성격의 메트릭

- 주요 메트릭
  - node_boot_time_seconds: 커널이 시작된 시간
  - node_intr_total: 누적된 하드웨어 인터럽트 개수
    - 높은 카디널리티 때문에 기본적으로 비활성화

### **uname 수집기**

> node_uname_info 게시

### **loadavg 수집기**

> 각각 1분, 5분, 15분 부하 평균을 의미하는 node_load1/5/15 메트릭 제공

- 리눅스에서는 실행 대기열에서 대기중인 프로세스의 개수 뿐 아니라 I/O를 기다리는 프로세스같이 인터럽트 할 수 없는 프로세스도 의미

### **textfile 수집기**

> 우리가 만든 파일에서 가져옴

- 커널에서 안가져옴

___

## 💿 **Service Discovery**

> 동적환경에서 추가되거나 삭제되는 머신 정보를 매번 prometheus.yml에 직접 반영하는 것은 매우 성가신 일이다.

- 즉 별도의 설정없이 Consul, EC2, K8s 같은 다양한 서비스 소스와의 연동을 지원
  - 지원하지 앟ㄴ으면 별도의 파일기만 서비스 검색을 통해 관련 정보를 획들할 수 있다.
  - 앤서블이나 셰프처럼 적절한 형식으로 머신과 서비스 목록을 작성해주는 환경 구성 관리시스템을 이용하
  - 데이터 소스에서 정기적으로 실행되는 스크립트를 작성하는 방법

 ### **메커니즘**

 > 서비스 검색은 머신 목록을 제공하거나 이를 모니터링하는 것만이 아니라 시스템 전반에 걸쳐 나타는 좀 더 일반적인 관심사항 같은 것

 - 따라서 머신들이 어떻게 구성되어 있으며 어떤 생명주기로 운영되는지에 대한 규칙도 확보
 - 메타데이터를 제공해야 한다.
   - 메타데이터: 서비스 이름, 설명 서비스 보유 팀, 구조화된 태그등
   - 메타데이터는 대상 레이블로 변환될 수 있으며, 일반적으로 메타데이터가 많을수록 더 좋다.

- 상향식: 컨설처럼 서비스 인스턴스를 등록할 때 메타데이터를 작성해 서비스 검색 메커니즘에 함께 등록하는 방식
  - 서비스 실행 시, 동기화를 위한 별도의 절충 과정이 필요할 수 있음
  - 따라서 App 인스턴스가 등록되기 전에 중단된 경우와 같은 상황을 감지
- 하향식: EC2 처럼 서비스 검색 메커니즘 스스로 구성 정보를 바탕으로 어떤 항목이 있어야 하는지 파악하고 접근하는 방식
  - 어떤 서비스가 실행될 예정이었지만 실제로 실행되지 않은 경우를 쉽게 감지

