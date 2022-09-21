---
title: (K8S) 레플리케이션 컨트롤러와 레플리카셋
author: simon sanghyeon
date: 2022-08-04
categories: [Kubernetes]
tags: [Kubernetes, K8S, Replicaset]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

# Replication Controller
`replication controller`는 쿠버네티스의 가장 기본적인 컨트롤러로, 파드가 항상 지정된 갯수만큼 클러스터 내에서 실행될 수 있도록 보장하는 역할을 한다.

예를 들어, 클러스터 내 파드가 2개 실행되어야 한다고 정의했는데, 그 중 하나가 장애로 인해 다운되어 1개만 실행된다고 가정해보자.
replication controller는 2개의 파드가 실행되어야 한다는 정의에 따라 재빨리 1개의 파드를 재실행하여 복구함으로써 2개의 파드가 실행되도록 보장해준다.

즉, replication controller는 쿠버네티스 클러스터 상에서 multiple instance를 재생할 수 있게 해줌으로써 고가용성(High Availability)이 가능하도록 지원하는 것이다.

## Replication Controller의 YAML
```yaml
# rc-definition.yaml

apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end

spec: # replication controller를 위해서 필요한 spec
  template:
    # 이곳에는 replication controller에 의해서 사용될 파드의 템플릿을 기입한다.
    # 파드 정의 YAML파일에서 apiVersion과 kind 빼고 복사 붙여넣기 해온다.
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec: # pod에 대한 spec
        containers:
        - name: nginx-container
          image: nginx

  replicas: 3 # how many replicas
```

replication controller 정의 YAML 파일을 위와 같이 작성한 후, `kubectl create -f rc-definition.yaml` 명령을 통해 생성할 수 있다.
또한 `kubectl get replicationcontroller` 명령을 통해 DESIRED 파드의 갯수, CURRENT 파드의 갯수, READY 상태 파드의 갯수 등을 확인할 수 있다.


---

# Replica Set
사실상 replication controller 보다는 더욱 최신 기능인 `replica set`을 더욱 많이 사용한다.
replica set은 replication controller와 거의 동일한 역할을 하되, 집합 기반(set-based)의 다양한 셀렉터 기능을 지원한다.
예를 들어, replica set의 셀렉터는 `in`, `notin`, `exists`와 같은 연산자를 지원한다.

## Replica Set의 YAML
```yaml
# replicaset-definition.yaml

apiVersion: apps/v1 # Replication Controller와 다른 점을 주의하자! apps/를 붙여야한다!
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end

spec: # replica set을 위해서 필요한 spec
  template:
    # 이곳에는 replica set에 의해서 사용될 파드의 템플릿을 기입한다.
    # 파드 생성에 필요했던 YAML파일에서 apiVersion과 kind 빼고 복사 붙여넣기 해온다.
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec: # pod에 대한 spec
        containers:
        - name: nginx-container
          image: nginx

  replicas: 3 # how many replicas

  # Replication Controller와 다른 점
  selector: # The selector section helps the replica set identify what pods fall under it.
    matchLabels:
      type: front-end
```

replica controller와 달리, replica set은 `selector`를 사용하는 것을 기억하자. 참고로, 아래와 같이 `matchExpressions`를 통해서 복잡한 selector 기능을 사용할수도 있다.

```yaml
...
spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [soaktestrs, soaktestrs, soaktest]}
      - {key: teir, operator: NotIn, values: [production]}
  template:
     metadata:
...
```

replica set 정의 YAML 파일을 위와 같이 작성한 후, `kubectl create -f replicaset-definition.yaml` 명령어로 생성할 수 있으며, `kubectl get replicaset`로 확인할 수 있다.

---
# Replicas의 숫자 변경하기
배포된 replica set의 replicas 숫자를 변경하는 방법은 아래와 같다.

- **방법 1** : YAML 파일에서 replicas 숫자를 바꾼 후 `kubectl replace -f replicaset-definition.yaml` 명령을 입력함
- **방법 2**
    - YAML을 이용하는 경우 : `kubectl scale --replicas=<바꾸고 싶은 숫자, 예를 들어 6> -f replicaset-definition.yaml`
    - 명령어를 이용하는 경우 : `kubectl scale --replicas=<바꾸고 싶은 숫자, 예를 들어 6> replicaset <metadata에 적은 type 이름, 예를 들어 myapp-replicaset>`
        - 이 경우 YAML 파일에 있는 내용이 바뀌는 것은 아니라는 점을 주의할 것

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
