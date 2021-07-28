---
title: Kubernetes network 2
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

**순서**
> 1. LoadBalancer가 원하는 node의 노출된 IP 와 노출된 매핑 포트(10.100.0.3:32213)로 요청을 보냅니다.
> 2. gateway는 service가 존재하는 node로 요청을 보냅니다.(eth0, 10.100.0.3)
> 3. netfilter는 kube-proxy로 목적지를 바꿉니다.
> 4. kube-proxy는 매핑된 포트번호(32213)을 보고 Cluster IP로 목적지를 바꿔서 목적지로 패킷을 전달합니다.

### **문제점**

1. NodePort 사용시 클라이언트 측에 non-standard 포트를 열어주어야함
   - 로드 밸런서를 통해 해결
   - 로드 밸런서에서는 일반 포트를 열어주고 실제 NodePort의 포트는 사용자로부터 안보이게 만들면 되기 때문
2. 요청자의 source IP를 의도치 않게 가린다는 제약이 있음

___

## 💿 **Load Balancer**

위에도 설명하였 듯이 모든 위부 트래픽은 NodePort를 통해 클러스터 내부로 들어온다. 로드 밸런서는 이에 더해 쿠버네티스가 이 모든 네트워크 컨트롤을 할 수 있게 한다.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: service-test
spec:
  type: LoadBalancer
  selector:
    app: service_test_pod
  ports:
  - port: 80
    targetPort: http
```

위와 같이 서비스를 생성하면 외부 공인 IP가 생성된다.

```bash
$ kubectl get svc service-test  
NAME      CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE  
openvpn   10.3.241.52     35.184.97.156   80:32213/TCP     5m
```

외부 공인 IP가 생성되기 위해서는 

1. forwarding rule 설정
2. target proxy
3. 백엔드 서비스와 instance group 
4. 외부 공인 IP

이 설정 및 생성 되어야 한다. 이러한 것들은 자동으로 해준다.

**제약사항**
- TLS termination 설정이 불가능 함
  - 따라서 한개의 로드 밸런서를 이용하여 여러 서비스에 연결을 하는 것이 불가능
  - TLS/ssl termination트래픽을 암호화 및 암호 해독하는 컴퓨팅 집약적인 작업으로부터 백엔드 서버를 해방하기 위한 작업
    - ex) client 와 lb 사이의 개방된 네트워크 에서는 https 통신, lb와 backend 사이는 http 통신을 가능케 하는 reverse proxy

![image](https://user-images.githubusercontent.com/15958325/89248733-6c888900-d64b-11ea-9cfd-6b2aa0df8ce0.png)
_tls/ssl termination_

위와 같은 제약사항을 해소하기 위해서 Ingress라는 서비스가 생겼습니다.

LoadBalancer 서비스 타입은 단지 한개의 내부 서비스를 외부 사용자들에게 접근 가능하도록 만드는 일을 담당합니다. 반대로 Ingress는 Tls termaintation이나 virtual hosts, path-based routing을 가능하게 합니다. 따라서 Ingress는 쉽게 한개의 로드 밸런서로 여러개의 backend 서비스들을 연결할 수 있게 합니다.

___

## 💿 **Ingress**

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  tls:
    - secretName: my-ssl-secret
  rules:
  - host: testhost.com
    http:
      paths:
      - path: /*
        backend:
          serviceName: service-test
          servicePort: 80
```

Ingress Controller는 위의 방식대로 들어오는 요청을 적절한 서비스로 전달하는 역할을 담당한다. Ingress를 사용할 때 요청 받을 서비스를 NodePort 타입으로 설정하고 Ingress-Controller로 하여금 어떻게 요청을 각 노드에 전달할지 파악하게 합니다. 각 클라우드 플랫폼 마다의 Ingress-Controller 구현체가 있습니다. GCP의 경우 cloud load balancer, AWS에서는 elastic load balance가 있고 오픈소스로는 nginx나 haproxy 등이 있습니다. 어떤 환경에서는 LoadBalancer와 Ingerss를 같이 쓰면 오류가 생긴다.


