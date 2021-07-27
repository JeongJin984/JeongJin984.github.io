---
title: Kubernetes network
author: Jiny
date: 2021-07-27 12:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes network

___

## 💿 **kubernetes**

### ** **Docker**

> A pod consists of one or more containers that are collocated on the same host, and are configured to share a network stack and other resources such as volumes.

여기서 중요한 점은 네트워크와 볼륨과 같은 특정 리소스를 공유한다는 것이다. 그렇다면 네트워크를 공유한다는 것은 무엇을 의미할까

![image](https://miro.medium.com/max/875/1*0Xo-WpbTTGKZhJt7TvFLZQ.png)

도커의 경우 docker0라는 veth0(172.17.0.2)를 위한 default gateway를 설정한다. 이 상태로 두번째 컨테이너를 띄우면

![image](https://miro.medium.com/max/875/1*ZdgIoY6tuOqK-r6wgL7d5A.png)

이 상태가 된다. 두 컨테이너가 통신하기 위해서는 두 컨테이너의 주소를 알고있는 docker0를 통해서 수행한다.

이렇듯 도커는 컨테이너를 시작할 때 새로운 network interface를 생성하는 대신 docker0 같은 공유 인터페이스를 이용한다.
  

### **K8s**

![image](https://miro.medium.com/max/875/1*7JLi1Rl0G0FAeu-hiTGSGQ.png)

2개의 컨테이너가 생성된 모습이다. 2개의 컨테이너는 172.17.0.2의 주소가 정해져 있으며 이 주소(네트워크)를 공유한다. 이 2개의 컨테이너는 port로 구별될 수 있다.(즉 내부에서 localhost와 port hit 될 수 있음) 이 상황은 single host에서 multiple process를 실행하는 것과 같다. 따라서 간단한 네트워크 환경에서 decoupling 과 컨테이너 고립의 장점을 얻을 수 있다.

쿠버네티스에서 pause 컨테이너는 Pod 내부의 컨테이너들을 위한 일종의 '부모 컨테이너' 로서의 역할을 수행합니다. 

pause 컨테이너는 주로 2가지 역할을 수행합니다. 
1. 첫 번째로, pause 컨테이너는 pod의 컨테이너들이 리눅스 namespace를 공유할 수 있도록 해줍니다. 
2. 두번째로, PID (Process ID) namespace 공유가 설정되었을 때에는, Pod에서 PID 1 (역주 : Init Process) 로서의 역할뿐만 아니라 좀비 프로세스를 거둬들이는 기능도 수행합니다. 

### **k8s Pod Network**

**K8s 네트워크 조건**
- 각각의 Pod은 전체 클러스터에서 유니크한 자신만의 IP를 가진다.
- 같은 노드에 있는 모든 Pod은 서로간에 통신이 가능해야 한다.
- 다른 노드에 있는 모든 Pod은 별도의 NAT 없이 통신이 가능해야 한다.

쿠버네티스는 이제 노드의 공통 Gateway를 통해 pod에서 발생된 packet의 목적지를 특정할 수 있게 되었다. 하지만 노드는 서로 독립된 네트워크 환경에서 작동하므로 docker0가 생성한 네트워크 환경이 겹칠 수 있는 여지가 충분히 있다. 그 경우가 아래의 그림이다.

![image](https://miro.medium.com/max/875/1*RiLtoAdCfcJygwePVJzZOA.png)

이 경우 목적지로 정해진 172.17.0.2로 설정된 네트워크가 다수 존재하므로 bridge에서 그 목적지를 특정할 수가 없다는 문제가 생긴다.

> 즉 one node typically has no idea what private address space was assigned to a bridge on another node

이를 해결하기 위해 2가지 해결법이 존재한다.
1. 각 노드의 bridge에 대한 전체 주소 공간을 할당하고 구축된 노드의 네트워크를 기반으로 해당 공간의 브리지 주소를 할당
   - 즉 node의 주소를 활용해서 Unique한 주소 공간을 할당하자
2. Routing rule을 node의 공통 Gateway에 추가하여 subnet-mask를 이용하여 패킷의 목적지에 해당하는 bridge의 주소를 연결하자


2번째 방법이 밑의 그림이다.

![image](https://miro.medium.com/max/875/1*oyGbXt7kStLd85ZT4it3oQ.png)

쿠버네티스는 birdge로 cbr(custom bridge)을 docker 대신 사용한다. 그 역할은 잘 모른다.

___

## 💿 **CNI(Common Network Interface)**

지금 까지 리눅스 브릿지를 이용하여 컨테이너 간의 통신을 가능케 하는 방법에 대해 알아봤다. 하지만 이러한 방법은 너무 다양하며 복잡하며 그리고 별도의 로직을 요구한다는 것이 단점이다.

그래서 탄생한것이 CNI인데 CNi란 Pod간의 통신에 규약(CNI)을 정하고 그 규약에 따라 만든 구현체를 CNI 플러그 인이라고 한다.

각각의 노드는 kubelet을 통해 노드를 제어한다. kubelet은  --cni-conf-dir=/etc/cni/net.d를 통해 전달된 디렉토리를 읽어 어떤 CNI를 사용할 지 결정한다. 또한 --cni-bin-dir=/opt/cni/bin를 통해 CNI 플러그인의 바이너리 위치를 전달한다.

### **Weave Net**

> CNI 플러그인 중 하나이다.

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcfvdhW%2FbtqDc5ijCEW%2F2HgDMh1Uh82WScMDP8yct0%2Fimg.png)

지금까지는 IP를 적당한 값으로 수동으로 할당했지만 실제 k8s 환경에서는 적절한 IP를 할당하고 관리해주는 IPAM(IP Adress Management)가 필요하다. 이 또한 CNI가 책임지고 있다. Weave 에서는 10.32.0.0/12의 대역 IP를 사용한다.

Weave에서는 먼저 각 노드에 적당한 Subnet을 할당하고 Bridge와 Pod이 해당하는 Subnet을 사용하게 만듭니다. 어떤 Subnet을 사용할지는 Weave에 의해 자동으로 정해지지만 설정을 통해 직접 지정할 수도 있습니다.

___

## 💿 **K8s Service**

Pod의 IP는 통신하기위한 엔드포인트로 적절하지 않다. 예를 들어 다수의 Pod를 Deployment로 구성하는 경우 Pod는 App의 부하 정도에 따라 스케일이 바뀐다. 즉 IP는 임시적이다. 따라서 영구적인 엔드포인트를 제공하기 위해 Service를 엔드포인트로 사용한다.Pod는 특정 라벨을 가지고 있고 그 라벨을 서비스가 바라보고 있다. 그래서 외부 패킷 Pod의 IP가 바뀌더라도 Service의 label를 통해 Pod에 접근할 수 있다.

### **기능**

1. LoadBalancer: 부하분산 장치
   - 쿠버네티스 외부에서 접근이 가능한 외부 부하 분산 기능
   - 내부 사용자만 접근 가능한 내부 부하 분산 기능이 있다.
2. Cluser IP : 서비스는 생성시 이름과 IP주소를 가집니다. 이 IP를 통해 Pod에 접근 가능한데 이 때 IP를 Cluster IP 라고 함
   - 불변성
3. Node Port: 클라이언트 IP 뿐만 아니라 클러스터를 구성하는 각 노드의 Port로 접속도 가능하다.
4. External Name
   - ExternalName은 외부 서비스를 쿠버네티스 내부에서 호출하고자할때 사용할 수 있다.
   - 쿠버네티스 클러스터내의 Pod들은 클러스터 IP를 가지고 있기 때문에 클러스터 IP 대역 밖의 서비스를 호출하고자 하면, NAT 설정등 복잡한 설정이 필요하다. 이때 서비스를 External 타입으로 설정하고 주소를 DNS로 설정하면 이 서비스는 들어오는 모든 요청을 DNS로 포워딩함

### **Kube Proxy**

클라이언트는 Service Port로 inbound 연결을 하고 proxy에서 서버로 outbound 연결을 시행함. 이런 종류의 proxy 서버들은 모두 user space에서 동작하기 때문에 모든 패킷들은 user space를 지나 kernal space를 거쳐 proxy된다. 이러한 작업을 위해서는 interface device가 필요한데 kube proxy 또한 user space proxy로 구성되어 있으므로 interface device는 host에 존재하는 ethernet interface이거나 Pod내에 존재하는 가상 ethernet interface 두개 뿐이다.

다만 이것들은 가상이므로 포트를 열고 커넥션을 맺는 등의 행위는 할 수 있지만 그 자체를 이용하지는 못합니다. 이를 해결하기 위해서 리눅스 커널 기능 중 하나인 netfilter와 user space에 존재하는 interface인 iptables라는 녀석들을 이용하여 해결합니다. netfilter는 간단히 말하자면 Rule-Based 패킷 처리 엔진이다. 즉 모든 오고 가는 패킷의 생명주기를 관리한다는 뜻이다.

그렇다. netfilter란 kernal space에 존재하는 kube proxy입니다.

아래 도표는 kube-proxy가 user space proxy로 실행될때 netfilter의 역할에 대해서 설명합니다.

![image](https://coffeewhale.com/assets/images/k8s_network/02_05.png)
  
1. kube-proxy가 localhost interface에서 service의 요청을 받아내기 위해 10400 포트(예제 기준)를 엽니다.
2. netfilter로 하여금 service IP로 들어오는 패킷을 kube-proxy 자신에게 라우팅 되도록 설정을 합니다.
3. kube-proxy로 들어온 요청을 실제 server Pod의 IP:Port로 요청을 전달합니다. (예제에서는 10.0.2.2:8080)

> 이러한 방법을 통해 service IP 10.3.241.152:80로 들어온 요청을 마법처럼 실제 server Pod가 위치한 10.0.2.2:8080로 전달할 수 있습니다. 

netfilter의 능력을 보자면, 이 모든 것을 하기 위해서는 단지 kube-proxy가 자신의 포트를 열고 마스터 api 서버로 부터 전달 받은 service 정보를 netfilter에 알맞는 규칙으로 입력하는 것 외엔 다른 것이 필요 없습니다.

k8s는 userspace에서 proxing을 수행하기 때문에 모든 패킷을 user space에서 kernal space로 변환해야 합니다. 이를 해결하기 위해 kubernetes 1.2 kube proxy에서는 iptables mode가 생겼습니다. 이 모드에서는 kube proxy가 직접 proxy의 설정을 하지 않고 그 역할을 netfilterdp 맡깁니다. 이를 통해 service IP를 발견하고 그것을 실제 Pod로 전달하는 것은 모두 netfilter가 담당하게 되었고 kube-proxy는 단순히 이 netfilter의 규칙을 알맞게 수정하는 것을 담당할 뿐입니다.

![image](https://coffeewhale.com/assets/images/k8s_network/02_06.png)

__

## 💿 **Kube Proxy의 내구성**

> Kube Proxy는 기본적으로 systemd unit으로 동작하거나 daemonset으로 설치가 됩니다. 따라서 프로세스가 죽어도 다시 살아날 수가 있습니다.

- 다만 kube proxy는 user space 모드에서 동작할때 단일 지점 장애점이 될 수 있습니다. 하지만 ip table 모드에서 실행 할 때는 안정적으로 작동합니다.
  - 왜냐하면 이는 netfilter를 통해 동작하고 서버가 살아있는 한 netfilter도 동작하는 것을 보장 받을 수 있기 때문입니다.

___

## 💿 **Kube Proxy의 Service 감지**

kube proxy는  마스터 노드의 api server의 정보를 수신하기 때문에 클러스터의 변화를 감지함 이를 통해 지속적으로 iptable을 업데이트하여 netfilter의 규칙을  변화시킵니다. 새로운 서비스가 생성되면 kube-proxy는  알림을 받고 그에 맞는 규칙을 생성합니다. 반대로 service가 삭제되면 이와 비슷한 방법으로 규칙을 삭제합니다. 

서버의 health-check는 kubelet을 통하여 수행합니다. kubelet은 서버에서 설치되는 쿠버네티스 컴포넌트 중 하나로 서버(노드의) Pod를 직접 관리합니다. 이 kubelet이 서버의 health check을 수행하여 문제를 발견시 마스터 api 서버를 통해 kube-proxy에게 알려 unhealthy Pod의 endpoint를 제거합니다.

이러한 것들을 기반으로 고가용성을 유지하는데 단점으로는 Pod에서 요청한 Request만 위와 같은 방법으로 동작합니다. 
또한 netfilter를 사용하기 때문에 외부에서 들어온 요청에 대해 원 요청자의 origin IP가 수정된다는 것입니다. 이 부분은 Ingress의 설명에서 깊게 설명하겠습니다.