---
title: (K8S) 쿠버네티스 API groups
author: simon sanghyeon
date: 2022-08-21
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, API groups, Certificate]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
쿠버네티스를 사용하면, kubectl 명령을 통해서 접근하든, 직접 접근하든 간에 kube-apiserver를 통해 접근해야만 한다. 예를 들어, 버전을 체크하는 명령을 수행한다고 해보자. `curl https://kube-master:6443/version` 과 같이 master 노드의 주소와 `기본 포트번호 6443`으로 apiserver에 접근할 수 있으며, API `/version` 통해 버전을 체크할 수 있다.

유사한 방식으로, 파드의 리스트를 보고자 한다면, `curl https://kube-master:6443/api/v1/pods` 와 같이 master 노드의 주소와 `기본 포트번호 6443`으로 접근하되, API `/api/v1/pods`를 씀으로써 접근할 수 있다. 그렇다면, 버전을 볼 때는 왜 `/version` 으로 접근하고, 파드를 볼 때는 `/api` 로 접근하는 것일까?

# 쿠버네티스 API의 다양한 그룹들
위 예시 이외에도, 쿠버네티스의 API는 목적에 따라서 다양한 그룹들이 존재한다. 그 종류는 다음과 같다.

- `/metric` : 클러스터의 상태를 모니터링하기 위한 API
- `/healthz` : 클러스터의 상태를 모니터링하기 위한 API
- `/version` : 클러스터의 버전을 확인하기 위한 API
- `/api` : 클러스터의 기능과 관련한 API (core group)
- `/apis` : 클러스터의 기능과 관련한 API (named group)
- `/logs` : 서드 파티 로깅 애플리케이션을 통합하여 사용하기 위한 API

이들 중 `/api`와 `/apis`를 살펴보고자 하는데, 전자인 `/api`는  `core group`으로, 핵심 기능들이 하위에 존재하며, 그 예로는 아래와 같다.

![fig01](/assets/img/2022-08-21-API Groups/fig01.png)

한편 후자인 `/apis`는  `named group`으로, 아래와 같이 좀더 계층적으로 구조화 되어 있다.

![fig02](/assets/img/2022-08-21-API Groups/fig02.png)

`/apis` 하위에는 `/apps` , `/extensions` , `/networking.k8s.io` , `/storage.k8s.io` , `/authentication.k8s.io` , `/certificates.k8s.io` 등으로 구성되어 있는 `API Groups`가 존재하며, 그 하위에는 `Resources`가, 그리고 Resource의 하위에는 각 Resource가 취할 수 있는 행동인 `Verbs`가 존재한다.

쿠버네티스에서도 위 구조들을 간단한 명령어로 확인할 수 있는데, `curl http://localhost:6443 -k` 와 같이 추가적인 path 없이 kube-apiserver에 접근하면 사용 가능한 API groups들을 확인할 수 있다. 또한 `curl http://localhost:6443/apis -k | grep "name"` 과 같은 명령어로 named group 중 사용 가능한 API group들을 확인해볼 수 있다.


# 쿠버네티스 API와 인증
![fig03](/assets/img/2022-08-21-API Groups/fig03.png){: width="500" height="500"}

`curl http://localhost:6443 -k` 명령어로 직접 kube-apiserver에 접근을 시도할 경우, 별도로 인증 메커니즘을 지정하지 않으면 접근이 제한될 수 있다. 이 경우, API에 인증서 파일(certificate file)을 명시함으로써 인증을 받을 수 있다. 그 예는 다음과 같다. `curl http://localhost:6443 -k --key admin.key --cert admin.crt --cacert ca.crt`

![fig04](/assets/img/2022-08-21-API Groups/fig04.png){: width="500" height="500"}

또 다른 방법으로는 `kubectl proxy` 명령을 통해서 proxy service를 local에서 포트번호 8001번으로 실행하고, kubeconfig 파일에 정의된 certificate를 이용하여 클러스터에 접근할 수 있다. 즉, `curl http://localhost:8001` 처럼 proxy service에 접근하면, proxy가 kubeconfig로부터 certificate를 이용해서 요청을 kube-apiserver에 전달해주는 것이다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
