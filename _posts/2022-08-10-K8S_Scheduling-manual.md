---
title: (K8S) 매뉴얼 스케줄링
author: simon sanghyeon
date: 2022-08-10
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

# nodeName으로 직접 노드를 지정하여 스케줄링
쿠버네티스에서는 파드 정의 YAML 파일에서 `nodeName`을 직접 기입하여 파드가 실행될 노드를 지정할 수 있다.
이는 추후 정리할 `nodeSelector`나 `nodeAffinity`에 비해 직관적이지고 단순한 방식으로, 일반적으로는 잘 사용하지 않는 방법이다.

우선, 파드 정의 YAML 파일을 보면서 nodeName을 직접 지정해보자.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
  nodeName: node02 # 아래 설명을 참고할 것
```

위와 같이 `nodeName` field가 "node02"처럼 직접 명시되어 있는 파드는 `스케줄러(scheduler)`가 해당 파드를 스케줄링 하지 않으며, nodeName에 명시된 노드(e.g., node02)의 `kubelet`이 해당 파드를 해당 노드에서 실행한다.

![fig01](/assets/img/2022-08-10-K8S_Scheduling-manual/fig01.png)

스케줄러는 nodeName이 설정되지 않은 파드를 발견하면, scheduling 알고리즘을 실행하여 파드를 위한 적합한 노드를 확인하고 스케줄링 한다.

## 스케줄러가 없다면 어떻게 될까?
파드의 STATUS가 지속적으로 Pending 상태로 남아있게 된다. 스케줄러가 없다면 우리가 수동으로 직접 파드를 노드에 assign 해야한다. 따라서 위와 같은 방식으로 파드 정의 YAML 파일에서 nodeName 필드를 직접 채워 넣어야만 하고, '파드가 생성된 때에만' 노드에 assign 될 수 있다.

## 파드가 이미 만들어져 있는 경우라면?
쿠버네티스가 파드의 nodeName을 바꾸지는 않는다. 이 경우에는 이미 있는 파드를 특정 노드로 assign하기 위해 새로운 방법을 쓰는데, `Binding`이라는 오브젝트를 만들어서 파드 binding API로 POST request를 보내는 방법이 있다. 스케줄러가 하는 것을 비스무레 따라하는 것이다.

---

# 매뉴얼 스케줄링의 문제
nodeName에 명시된 노드가 없는 경우, 파드는 실행되지 않고 자동으로 지워질 수 있다. 또한 nodeName에 명시된 노드에 파드가 실행될 리소스가 충분하지 않다면, 파드 실행이 실패할 것이다. 마지막으로, 클라우드 환경에서 nodeName 자체가 항상 동일하지 않으므로 안정적이지 않다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `쿠버네티스 공식 documentation` : [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename)
