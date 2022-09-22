---
title: (K8S) 노드 셀렉터와 노드 어피니티
author: simon sanghyeon
date: 2022-08-15
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling, NodeSelector, NodeAffinity]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

이번 글에서는 파드를 노드에 배치하는 방법으로 `노드 셀렉터(nodeSelector)`와 `노드 어피니티(Node Affinity)`를 정리한다.
nodeName을 이용하는 [매뉴얼 스케쥴링](https://zerojsh00.github.io/posts/K8S_Scheduling-manual/) 및 [테인트와 톨러레이션](https://zerojsh00.github.io/posts/K8S_Taints-and-Tolerations/)을 함께 참고하면 좋겠다.

# 파드 배치의 필요성

![fig01](/assets/img/2022-08-15-Node Selector and Node Affinity/fig01.png)

위와 같이 클러스터 내에서 노드들의 리소스는 각각 다를 수 있다. 예를 들어, 어떤 노드는 다른 노드보다 더욱 고성능의 하드웨어 리소스를 보유할 수 있는 것이다. 이러한 상황에서 default 세팅으로 파드를 배치하면 다음과 같은 문제가 발생하게 된다.

![fig02](/assets/img/2022-08-15-Node Selector and Node Affinity/fig02.png)

즉, 파드B와 같이 많은 하드웨어 리소스를 소요하는 파드가 노드02와 같이 저성능 노드에 배치되어 파드를 실행하기 위한 하드웨어 자원이 부족해지는 현상이 발생하는 것이다.

이외에도 하드웨어의 리소스를 고려하여 특정 파드를 특정 노드에서 실행되도록 해야 하는 상황은 다양하게 있을 수 있다.
예를 들면, 어떤 파드는 반드시 SSD에 연결된 노드에 배포해야 하는 경우가 있다.
이러한 경우에 어떻게 특정 파드를 특정 노드에 배포할 수 있는지 살펴보자.

---

# 노드 셀렉터(nodeSelector)

`노드 셀렉터(nodeSelector)`는 파드가 특정 노드에서 실행되도록 하는 가장 쉬운 방법 중 하나로, **nodeSelector는 파드 정의 YAML 파일과 노드에서 사용되는 일종의 label**이다.

## 사용법 step 01 : 파드 정의 YAML 파일에 nodeSelector 레이블을 설정함

```yaml
# pod-definition.yaml

kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor

  nodeSelector: # 이 부분을 통해서 특정 노드에서만 파드가 실행되도록 한다.
    size: Large
    # 사실, 이 size: Large는 노드에 부여되는 '레이블'이다.
    # 스케줄러는 size: Large라고 assign된 레이블이 match되는 노드에 이 파드를 배치한다.
    # 따라서 이 파드를 배치하기 전에 노드에 size: Large라는 레이블이 먼저 부여되어 있어야 한다.
```

## 사용법 step 02 : 파드가 실행될 노드에 동일하게 레이블을 설정함

파드에 설정된 nodeSelector는 단지 레이블일 뿐이기 때문에, 노드에도 레이블을 달아주어야 한다.
노드에 있는 레이블과 일치하는 경우에 이 파드를 해당 노드에 배치할 수 있다.

`kubectl label nodes <노드 이름> <레이블-키 값>=<레이블-밸류 값>`의 형태로 노드에 레이블을 달 수 있다.
예를 들면, `kubectl label nodes node-1 size=Large`처럼 말이다.

이후 `kubectl create -f pod-definition.yaml` 명령어를 통해 파드를 생성하면, 자동으로 레이블이 matching되는 노드에서 파드가 실행된다.

## 노드 셀렉터의 문제점

노드 셀렉터를 이용하는 것은 위 예제에서 살펴보았듯 `size: Large`와 같이 단 하나의 레이블만 사용할 수 있다.
만약, 더 복잡한 조건으로 노드를 선택해야 한다면 어떨까? 예를 들어, `size: 크거나 중간`과 같은 복잡한 조건이 들어가는 경우 말이다.
이땐 nodeSelector 레이블만으로 노드를 선택할 수는 없다. 즉, 노드 어피니티가 필요하다.

---

# 노드 어피니티(Node Affinity)

`노드 어피니티(Node Affinity)`는 앞서 살펴보았던 노드 셀렉터와 유사하게, 파드가 특정 노드에서만 실행될 수 있도록 설정하는 방법이다.
하지만 **노드 어피니티는 nodeSelector 레이블과는 달리, 레이블 선택에 있어서 'or' 조건이라든지 'not' 조건과 같이 복잡한 조건도 처리**할 수 있다.

## 사용법

```yaml
# pod-definition.yaml

kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor

  # nodeSelector   ... 노드 셀렉터와의 차이점
  # - size: Large   ... 노드 셀렉터와의 차이점

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: # 노드 어피니티의 속성
      - matchExpressions:
        - key: size
          operator: In # NotIn, Exsists 등도 가능
          values:
          - Large # 추가적으로 다양한 밸류값 가능
          - Medium
          - ...
```

노드 어피니티의 특징을 관찰해보자. 파드가 특정 노드에서 실행되도록 한다는 점에서 개념적으로는 노드 셀렉터와 동일한 기능이지만, 노드 셀렉터보다 훨씬 복잡하게 정의되어야 한다.

복잡하지만, 노드 어피니티는 `operator` 부분을 다양하게 설정하여 복잡한 조건들을 반영하면서 노드를 선택할 수 있게 한다.
심지어 values 값을 모를 때는 Exists라는 operator를 사용하고 values를 생략할 수도 있다.
참고로, `values` 값에는 더욱 다양하게 값들을 여러 개를 나열할 수 있다.

## 노드 어피니티의 속성 : matchExpression에 대응되는 노드가 없는 경우
만약, 노드 어피니티를 위해 파드 정의 YAML 파일에서 설정한 `matchExpression`에 대응되는 노드가 없다면 어떻게 될까? 또는, 미래에 누군가가 노드의 레이블을 바꾸어버린다면, 위와 같은 설정으로 만들어진 파드는 계속해서 해당 노드에서 실행될까? 이러한 이슈는 `requiredDuringSchedulingIgnoredDuringExecution`처럼 긴 속성으로 해결할 수 있다.

현재 지원되는 노드 어피니티의 속성 값은 아래와 같다.
- <span style='background-color: #FFCC4E'>required</span><span style='background-color: #D5E05B'>DuringScheduling</span><span style='background-color: #B0DFDB'>Ignored</span><span style='background-color: #BBB8DC'>DuringExecution</span>
- <span style='background-color: #81D3EB'>preferred</span><span style='background-color: #D5E05B'>DuringScheduling</span><span style='background-color: #B0DFDB'>Ignored</span><span style='background-color: #BBB8DC'>DuringExecution</span>

### DuringScheduling
<span style='background-color: #D5E05B'>DuringScheduling</span>이란, 아직 파드가 존재하지 않는 상태이며, 처음으로 파드가 생성되는 시점을 의미한다.

속성이 <span style='background-color: #FFCC4E'>required</span>로 설정되어 있다면, 파드를 노드에 배치함에 있어 `어피니티 규칙(affinity rule)`에 의한 matching이 필수인 경우를 의미한다.
즉, 스케줄러는 파드가 처음 생성될 때 어피니티 규칙에 따라 노드에 배치되도록 지시하는데, 만약 어피니티 규칙에 따라 matching되는 노드가 없으면, 파드는 스케줄링되지 않는다.
이는 파드가 특정 노드에 배치되도록 설정하는 것이 매우 중요한 상황에서 사용된다.

속성이 <span style='background-color: #81D3EB'>preferred</span>로 설정되어 있다면, 파드를 노드에 배치함에 있어 우선적으로 어피니티 규칙을 고려하되, 어피니티 규칙에 matching되는 노드가 없는 경우에는 아무 노드에 배치될 수 있다. 이는 파드가 특정 노드에 배치되도록 설정하는 것이 선호되기는 하지만 필수는 아닌 상황에서 사용된다.

### DuringExecution
<span style='background-color: #BBB8DC'>DuringExecution</span>이란, 파드가 이미 만들어져 있고 특정 노드 상에서 실행 중에 있는데, 노드의 레이블이 변경되는 상황과 같이 노드 어피니티에 영향을 주는 변화가 있는 시점을 의미한다.
예를 들어, 실행 중인 파드는 어피니티 규칙에 의해 `size:Large`로 레이블된 노드에서 실행되고 있는데, 노드의 레이블이 바뀌면 파드는 계속 실행될 수 있는 것인지를 이 부분에서 설정한다.

속성이 <span style='background-color: #B0DFDB'>Ignored</span>로 설정되어 있기 때문에 노드 어피니티에 영향을 주는 변화가 있어도 파드는 이를 무시하고 계속해서 실행된다.

(참고로, 현재까지 쿠버네티스에서 지원되는 속성들은 모두 <span style='background-color: #B0DFDB'>Ignored</span>이지만, 추후 `Required` 등도 지원될 예정이다. `requiredDuringSchedulingRequiredDuringExecution`)

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `쿠버네티스 공식 documentation` : [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename)
