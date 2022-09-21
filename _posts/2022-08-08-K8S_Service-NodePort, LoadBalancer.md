---
title: (K8S) 서비스(Service) 타입 - NodePort & LoadBalancer
author: simon sanghyeon
date: 2022-08-08
categories: [Kubernetes]
tags: [Kubernetes, K8S, Service]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

앞서 쿠버네티스 환경에서 [서비스(Service)의 개요와 종류](https://zerojsh00.github.io/posts/K8S_Service/)를 간단히 살펴보았다.
이번에는 서비스의 종류 중 `NodePort` 및 `LoadBalancer` 타입의 서비스를 살펴보고자 한다.

ClusterIP에 대한 포스트는 [여기](https://zerojsh00.github.io/posts/K8S_Service-ClusterIP/)를 참고하자.

# NodePort 서비스란?

![fig01](/assets/img/2022-08-08-K8S_Service-NodePort,%20LoadBalancer/fig01.png)

`NodePort` 서비스 타입은 '노드 포트'라는 명칭대로, 노드의 특정 포트(그림에서 30008)를 개방해 서비스에 접근하는 방식이다.

NodePort 서비스 타입은 서비스를 중심으로 총 3개의 포트를 사용한다.
- `Port` : 서비스의 포트(즉, cluster IP의 포트)
- `Node Port` : 노드의 포트
    - 별도로 지정하지 않으면 30000 ~ 32767번 사이의 포트가 자동으로 지정된다.
- `Target Port` : 최종 목적지인 파드의 포트
    - 별도로 지정하지 않으면 Port와 같은 값으로 지정된다.

## NodePort 정의하기

```yaml
# service-definition.yaml

apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec: # 가장 중요한 부분
  type: NodePort # 어떤 타입인지, NodePort, ClusterIP, LoadBalancer
  ports: # 서비스의 포트 관점에서
  - targetPort: 80
    port: 80
    nodePort: 30008
```

위의 예제에서는 targetPort를 정의하긴 했으나, 어떤 파드의 포트가 targetPort인지는 지정하지 않았다.
웹 서비스를 하면 80포트를 사용하는 파드가 수백개가 될 수도 있는데, 어떤 것을 의미하는 것일까?

특정 파드의 포트라는 것을 지정하기 위해서, ReplicaSet에서 했던 방식처럼 `selector`를 사용한다!

```yaml
service-definition.yaml

apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec: # 가장 중요한 부분
  type: NodePort # 어떤 타입인지, NodePort, ClusterIP, LoadBalancer
  ports: # 서비스의 포트 관점에서
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector: # 파드 생성 시, metadata의 labels로 정의했던 내용들을 아래에 써준다.
    app: myapp
    type: front-end
```
이 경우, 파드 생성 시 metadata 내 labels에 기입했던 정보를 selector에 써준다. 그러면 수많은 파드 중 해당 파드가 targetPort로 지정되는 것이다.
이후 , `kubectl create -f service-definition.yaml` 명령으로 서비스를 생성하며, `kubectl get services`로 서비스를 확인할 수 있다.

위와 같이 포트를 노출해준 후 서비스를 생성해주면, 클러스터 외부에서 노드의 특정 포트(e.g., 30008)를 경유하여 내부 파드로 접근할 수 있게 된다. 예를 들어, `curl http://192.168.1.2:30008`와 같이 확인해볼 수 있다.

## selector를 사용해도 동일한 내용의 파드가 여러 개 존재한다면?
위와 같이 `selector` 섹션에서 파드의 labels를 지정하여 특정 파드의 포트와 매핑하였지만, 이러한 파드 자체가 여러 개 존재한다면, 어떤 파드에 load를 부여할지 어떻게 선택할까?

- (Case1) **하나의 노드**에 있는 labels가 동일한 파드들이라면, random 알고리즘으로 그냥 무작위로 파드를 선택해서 연결한다.
- (Case2) **여러 개의 노드**에 있는 labels가 동일한 파드들이라면, 쿠버네티스는 여러 노드에 걸쳐있는 서비스를 하나 생성한다. 그리고 그 서비스는 클러스터 내 각 노드에 있는 모든 `targetPort`(e.g., 80)들과 각 노드의 동일한 포트, 즉 `nodePort`(e.g., 30008)를 연결한다.
    - 아래의 경우가 모두 작동하게 될 것이다.
        - `curl http://192.168.1.2:30008`
        - `curl http://192.168.1.3:30008`
        - `curl http://192.168.1.4:30008`

---

# LoadBalancer 서비스란?
`LoadBalancer` 타입의 서비스는 서비스 생성과 동시에 로드밸런서를 새롭게 생성해 파드와 연결한다.
NodePort를 사용할 때는 각 노드의 IP를 알아야만 파드에 접근할 수 있었으나, LoadBalancer 타입의 서비스는 클라우드 플랫폼(GCP, AWS, Azure)으로부터 도메인 이름과 IP를 할당받기 때문에 NodePort보다 더욱 쉽게 파드에 접근할 수 있다.
단, LoadBalancer 타입의 서비스는 로드 밸런서를 동적으로 생성하는 기능을 제공하는 환경만 사용할 수 있다. 일반적으로 AWS, GCP, Azure 등과 같은 클라우드 플랫폼 환경에서만 사용할 수 있으며, 가상 머신이나 온프레미스 환경에서는 사용하기 어려울 수 있다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
