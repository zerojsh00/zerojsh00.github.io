---
title: (K8S) 멀티 컨테이너 파드(Multi Container Pods)
author: simon sanghyeon
date: 2022-08-19
categories: [Kubernetes]
tags: [Kubernetes, K8S, Pod, Multi Container Pods]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 멀티 컨테이너 파드
## Monolithic 구조와 MicroService 구조 개념

![fig01](/assets/img/2022-08-19-Multi Container Pods/fig01.png){: width="500" height="500"}

대규모 `monolithic` 애플리케이션을 `microservice`로 불리우는 subcomponent들로 분리할 수 있다는 아이디어 덕에, 각각 독립적이며 작고 재사용 가능한 코드들을 개발하고 배포할 수 있게 되었다.

![fig02](/assets/img/2022-08-19-Multi Container Pods/fig02.png){: width="500" height="500"}

이러한 microservice 구조를 이용하면, 수정 사항이 있을 때 애플리케이션 전체를 수정해야하는 monolithic 구조와 달리 각각의 작은 서비스들을 수정할 수 있으며, 손쉬운 scale up 또는 scale down이 가능해진다.

예를 들어서, 하나의 파드 내에 logging을 담당하는 서비스와 웹 서버 서비스가 함께 작동해야 한다고 해보자. 이 두 서비스는 각각의 역할이 분명히 정해져 있기 때문에, logging 서비스와 웹 서버 각 컨테이너가 따로 개발되고 배포되어야 한다. 다만, 각 서비스들은 반드시 함께 작동되어야 한다.

이처럼 각각 독립적이지만 반드시 함께 작동해야 하는 서비스들이 한 데 묶여 함께 생성되기도 하고(scale up) 삭제되기도 하면서(scale down) 동일한 라이프 사이클을 가져야 하기 때문에 `멀티 컨테이너 파드(Multi-container Pods)`가 필요하다.

## 멀티 컨테이너 파드의 개념

![fig03](/assets/img/2022-08-19-Multi Container Pods/fig03.png){: width="200" height="200"}

멀티 컨테이너 파드 안에 있는 각 서비스(컨테이너)들은 동일한 네트워크를 공유하기 때문에 localhost로 서로 간 통신될 수 있으며, 동일한 스토리지 볼륨을 사용하므로 모두가 동일하게 접근할 수 있다.

## 멀티 컨테이너 파드의 정의

멀티 컨테이너 파드를 정의하는 방법은 매우 간단하다. 파드 생성 YAML 파일에서 컨테이너 정보를 입력하는 `spec` 부분은 리스트 형태로 되어 있는데, 단순히 리스트의 원소로 컨테이너의 정보를 입력하며 나열해주면 된다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers: # 아래 부분에서 컨테이너 정보들을 '-'를 사용하여 리스트 형태로 나열한다.
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
  - name: log-agent
    image: log-agent
```

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>

