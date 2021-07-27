---
title: Kubernetes network
author: Jiny
date: 2021-07-27 14:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes network 2

___

## 💿 **라우팅 & 로드 밸런싱**

네트워크의 traffic은 기본적으로 연결(connections) 과 요청(request)로 이루어져 있다. 이러한 traffic은 osi 4(tcp)나 osi 7(http, rpc, etc)에서 동작한다. netfilter의 Routing Rule은 IP Packet 레벨인 layer 3에서 동작한다.(netfilter를 포함한 모든 라우터는 라우팅 결정을 IP Packet을 기준으로 하므로)

내부에서 외부로 가는 패킷과 외부에서 내부로 오는 패킷은 똑같이 라우팅 방식을 이용하여 전달 된다. Service의 ClusterIP가 각 포드의 앞단에 위치하여 Client가 각 노드의 Pod의 주소를 몰라도 label을 이용하여 해당 Pod로 연결해주기 때문에 외부 클라이언트는 반드시 ClusterIP를 사용해서 Pod에 접근해야 합니다. 하지만 Cluster IP는 클러스터 내부에서만 사용할 수 있습니다. 외부에서는 해당 주소의 대역 밖에 확인되지 않습니다.

이 문제를 해결하기 위하여 첫번째로 생각할수 있는 것은 외부에서 접근 가능한 Service IP(Cluster IP를 netfilter를 고려 후 전달하여 그를 Service IP로 사용)를 설정하는 것입니다. 적절하게 사용자 친화적인 도메인 이름을 붙이고 해당 패킷이 어느 노드로 전달되어야 하는지에 대한 규칙과 함께 제공하는 겁니다.

이러면 클라이언트에서 서비스 IP를 호출하면 정의된 Routing Rule에 따라 특정 노드에 전달되고 그 노드의 네트워크 interface 에서는 netfilter를 사용하여 원하는 Pod로 전달된다. 이 방법의 문제점은

- 노드도 대체 가능한 자원(ephemeral)이다.
  - 새로운 VM으로 대체되거나 Scale Up/Down 할 수 있다.
  - 즉 노드가 더이상 네트워크 내에 존재하지 않는다면 문제가 생긴다. 
  - 노드가 유지된다 하더라도 모든 트래픽이 한 노드에 집중될 것이므로 이것은 최적의 선택이 될 수 없다.

이제 우리는 해결방법을 찾아야 한다. 하지만 단순히 라우터를 이용해서는 이러한 문제를 해결할 수 없다. k8s는 클러스터 외부에 존재하는 라우터를 관리하는데까지 역할을 넓히지 않았기 때문입니다. 이는 클라이언트의 요청을 분산하여 노드들에게 전달해주는 존재가 이미 존재하기 때문인데 이를 로드 밸런서라고 부릅니다.

로드 밸런서를 이용하여 클라이언트의 트래픽을 분산시키기 위해서는 

1. 클라이언트가 접속할 수 있는 공인 IP가 필요함 
2. 로드밸런서에서 각 노드로 트래픽을 분산 시키고 위해서 노드의 IP도 필요합니다.
3. 외부에서 들어온 IP를 Service로 연결 시키는 서비스

3번을 자세히 설명하자면 만약 로드밸런서에서 노드로 각 노드들의 ethernet interface인 10.100.0.0/24 대역을 사용하여 요청을 보냈다면 포트를 이용하여 보내야 되는데(10.100.0.3:80)이러면 에러가 발생한다.

![image](https://coffeewhale.com/assets/images/k8s_network/03_04.png)

netfilter는 들어온 요청을 일정한 규칙에 따라 Service의 IP로 바꿔주는 역할을 하는데 node의 주소는 맵핑이 되어있지 않다.(netfilter는 Pod끼리의 통신에서 Pod에서 발생된 패킷의 주소를 Routing Rule에 의해 변한하여 도착해야 하는 Service의 주소로 변환하고 그 주소는 Pod의 네트워크 대역에 따른 node의 위치가 매핑된 Gateway에서 처리되어 원하는 노드 -> Service로 이동한다.) 즉 netfilter는 Service의 IP만 바라보며 변화를 감지한다는 것이다.

이것은 즉 

- netfilter에 의해서 동작하는 네트워크는 노드 네트워크에서 잘 동작하지 않는다
- 반대로 노드 네트워크에서 동작하는 네트워크는 netfilter에 의해 패킷이 전달되지 않는다.

이 문제를 해결하기 위해서는 이 별도의 2개의 네트워크를 연결해 주는 bridge를 사용해야 한다. 그것이 **Nodeport**이다.

___

## 💿 **NodePort Service**

> NodePort 타입 서비스는 노드 네트워크의 IP를 통하여 접근을 할 수 있을 뿐만 아니라 ClusterIP로도 접근이 가능

![image](https://coffeewhale.com/assets/images/k8s_network/03_05.png)

- NodePort 타입의 서비스를 생성하면 kube-proxy가 각 노드의 eth0 네트워크 interface에 30000-32767 포트 사이의 임의의 포트를 할당하기 때문
- 할당된 포트로 요청이 오게 되면 이것을 매핑된 ClusterIP로 전달



