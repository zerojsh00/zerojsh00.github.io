---
title: (K8S) 데몬셋(DaemonSet)
author: simon sanghyeon
date: 2022-08-18
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling, DaemonSet]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 데몬셋(DaemonSet)이란?

![fig01](/assets/img/2022-08-18-DaemonSet/fig01.png)

`데몬셋(DaemonSet)`은 클러스터 내의 **모든 노드에 파드 복제본을 단 하나씩 존재하도록 보장**한다.
여러 개의 파드 복제본을 클러스터 내의 여러 노드에 배포한다는 점에서 레플리카셋과 유사하지만, 데몬셋은 각 노드마다 단 하나의 파드만을 배포한다는 차이가 있다.

클러스터 내 새로운 노드가 생겨나면, 데몬셋은 해당 노드에도 파드를 자동으로 배포한다.
마찬가지로 클러스터 내에 존재하던 노드가 사라지면, 그 노드에 데몬셋에 의해 배포되어 있던 파드를 자동적으로 제거(garbage collected)한다.

---
# 데몬셋의 활용

![fig02](/assets/img/2022-08-18-DaemonSet/fig02.png)

데몬셋의 전형적인 활용 예로는 모든 노드에서의 `클러스터 스토리지 데몬(daemon)`, `로그 수집 데몬`, `노드 모니터링 데몬`을 예로 들 수 있다.
이외에도 모든 워커 노드에 반드시 존재해야 하는 `kube-proxy`나 네트워킹 솔루션인 `weave-net` 등 역시 데몬셋의 중요한 활용 예시다.

---
# 데몬셋을 생성하는 방법

데몬셋은 레플리카셋과 생성하는 방식이 매우 비슷하다.
```yaml
# daemon-set-definition.yaml

apiVersion: apps/v1
kind: DaemonSet # 사실상 이 부분만 ReplicaSet과 다르며, 나머지는 동일하다.
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```
이후 `kubectl create -f daemon-set-definition.yaml` 명령을 통해서 생성할 수 있고, `kubectl get demonsets` 명령과 `kubectl describe daemonsets <daemonset의 이름>`을 통해서 생성된 데몬셋의 정보를 확인할 수 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
