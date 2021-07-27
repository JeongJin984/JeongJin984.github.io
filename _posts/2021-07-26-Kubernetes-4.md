---
title: Kubernetes 4
author: Jiny
date: 2021-07-26 12:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes 4

___

## 💿 **포드의 문제점**

![image](https://miro.medium.com/max/875/1*oyGbXt7kStLd85ZT4it3oQ.png)

- 포드는 일시적으로 생성한 컨테이너 집합
  - 때문에 포드가 지속적으로 생겨났을 때 서비스를 하기에 적합하지 않음
- IP 주소의 지속적인 변동, 로드밸런생을 관리해줄 또 다른 객체가 필요
- 이 문제를 해결하기 위해서 서비스라는 리소스가 존재

### **서비스**

- 서비스의 세션 고정
  - 서비스가 다수의 포드로 구성하면 웹서비스의 세션이 유지되지 않음
  - 이를 위해 처음 들어왔던 클라이언트 IP를 그대로 유지해주는 방법이 필요
  - sessionAffinity: ClientIP라는 옵션을 주면 해결 완료

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - port: 80
      targetPort: 9376
  sessionAffinity: ClusterIP
```

- 다중 포트 서비스
  - 여러 포트를 열어서(다양한 프로토콜) 서비스 제공 가능

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

___


## 💿 **Service 노출**

- NodePort: 노드 자체 포트를 사용하여 포드로 리다이렉션
- LoadBalancer: 외부 게이트웨이를 사용하여 노드 포트로 리다이렉션
  - 자바로 구현 가능한 듯?
- Ingress: 하나의 IP 주소를 통해 여러 서비스를 제공하는 특별한 메커니즘
  - Nginx

### **NodePort**

> 노드의 포트를 노출하여 포드로 리다이렉션

![image](https://www.polarsparc.com/xhtml/images/Kubernetes-1.png)

- kubectl을 통해 개발자가 api-server에 요청을 보내면 kubelet으로 요청이 전달되고 kubelet이 포드를 컨트롤 한다.
- 그리고 사용자들이 포드로 제공된 서비스를 이용하기 위해서 노출된 주소 및 포트로 노드에 요청을 보내면 kube-porxy로 전달되어 이를 통해 포드를 사용함

> 다만 이 둘의 요청은 둘다 node로 들어가기 위해서 노출된 노드의 포트를 사용한다는 공통점이 있음

### **LoadBalancer**

![image](https://www.alcide.io/wp-content/uploads/2021/01/image-16.png)

> L4 장비(외부 게이트웨이)를 사용하여 노드 포트로 리다이렉션

- 클러스터에 로드밸런싱을 해주는 개체 필요
- 로드밸런스는 노드포트의 기능을 포함
- 노출된 하나의 IP를 사용해서 개발자 및 사용자가 Pod에 접근 가능
- 부하 분산도 가능함

### **Ingress**

![image](https://banzaicloud.com/blog/k8s-ingress/ingress-host-based_huceb35be881c2eb47b69d4d146880e204_114327_960x0_resize_box_2.png)

> 하나의 IP나 도메인으로 다수의 서비스를 제공

- Host 부분을 해석하여 규칙에 따라서 Query Resource를 활용하여 서로다른 서비스 사용 가능 