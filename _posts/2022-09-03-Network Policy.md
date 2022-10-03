---
title: (K8S) 네트워크 정책(Network Policy) 기초 개념
author: simon sanghyeon
date: 2022-09-03
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Network]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 네트워크 트래픽에서의 Ingress와 Egress 개념
네트워크 트래픽은 `ingress`와 `egress`로 구분할 수 있다. ingress란 외부로부터 서버 내부로 유입되는 네트워크 트래픽을 의미하고, egress는 서버 내부에서 외부로 나가는 트래픽을 의미한다.

---
# 쿠버네티스의 “All Allow” Rule
쿠버네티스 클러스터 내에 있는 파드들은 기본적으로 상호 간 communication이 가능하도록 되어있다.
쿠버네티스의 default 네트워크 설정인 `all allow`는 특정 파드로부터 다른 어떠한 파드나 서비스로든 네트워크 트래픽이 전송되도록 하는 규칙이다.

---
# Network Policy
![fig01](/assets/img/2022-09-03-Network Policy/fig01.png){: width="500" height="500"}

위 그림과 같이 웹 서버에 대한 파드와 백 엔드 서버에 대한 파드, 그리고 데이터베이스 서버에 대한 파드가 있다고 예를 들어보자. 앞서 언급했듯, 기본적으로 쿠버네티스는 클러스터 내 모든 파드들끼리 서로 communication이 가능해야 하므로, 위의 그림과 같이 표현될 수 있다.

그러나, 웹 서버 파드가 데이터베이스 서버 파드와 직접적으로 communication 할 필요가 없는 경우를 고려하면, 위와 같이 모든 파드가 연결되어 있는 네트워크 설정을 원하지 않을 수 있다. 이때, 데이터베이스 서버는 백 엔드 서버 파드로부터의 ingress 트래픽만 허용되고, 웹 서버 파드로부터의 ingress 트래픽은 허용되지 않도록 설정할 수 있다. 바로, 레플리카셋에서 다루었던 `label`과 `selector`를 이용하는 것이다.

![fig02](/assets/img/2022-09-03-Network Policy/fig02.png){: width="500" height="500"}

아래와 같이 `네트워크 정책(Network Policy)`을 정의할 수 있다.

```yaml
# policy-definition.yaml

apviVersion: networking.k8s.io/v1
kind: NetworkPolicy # network policy도 파드나 ReplicaSet처럼 쿠버네티스의 오브젝트다.
metadata:
  name: db-policy
spec:
  podSelector: # 아래 레이블에 매치되는 파드(DB 파드)에 대해서 적용한다.
    matchLabels:
      role: db
  policyTypes: # ingress 트래픽에 대해 규칙을 정의한다.
  - 2022-09-17-Ingress
  ingress:
  - from: # 아래 레이블에 매치되는 파드(api-pod, 즉 백 엔드 파드)로부터 들어오는 ingress 트래픽
    - podSelector:
	      matchLabels:
	        name: api-pod
    ports:
    - protocol: TCP
      port: 3306 # 허용하고자 하는 파드 포트
```

이후, `kubectl create -f policy-definition.yaml` 명령을 통해서 규칙을 생성한다.

---
# 주의할 점
network policy는 `Kube-router`, `Calico`, `Romana`, `Weave-net` Container Network Interface(CNI)에서만 지원되며, `Flannel` 환경에서는 지원되지 않는다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
