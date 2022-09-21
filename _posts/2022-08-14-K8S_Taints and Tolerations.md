---
title: (K8S) 테인트와 톨러레이션
author: simon sanghyeon
date: 2022-08-14
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling, Taint, Toleration]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

이번 글에서는 어떤 파드를 어떤 노드에 배치하는지, 즉 파드와 노드 간의 관계를 다룬다.

# 사람에게 들러붙으려는 벌레로 살펴보는 예

테인트(taint)와 톨러레이션(toleration)의 관계는 처음 접할 때 다소 복잡할 수 있다. 따라서 사람에게 접근하는 벌레의 예를 들어서 설명하고자 한다.

- 벌레가 사람에게 오는 것을 막기 위해서 사람들은 스프레이를 뿌리는데, 이 스프레이를 `테인트`라고 생각하자.
- `벌레 A`는 스프레이 냄새를 굉장히 싫어하기 때문에, 사람에게 접근하면 스프레이(즉, 테인트) 냄새를 맡고 도망가게 된다.
- 한편 `벌레 B`는 스프레이 냄새를 버틸 수(tolerant) 있기 때문에 사람에게 접근할 수 있다.

위 예에서 보듯, 벌레가 사람에게 들러붙을 수 있으려면 두 가지가 만족되어야 한다.
1) 사람에게 스프레이(즉, 테인트)가 얼마나 뿌려져 있는가?
2) 벌레는 얼마나 스프레이(즉, 테인트) 냄새를 버틸 수 있는가? (toleration)

쿠버네티스에 적용해보면, `사람 = 노드`이며, `벌레 = 파드`로 비유할 수 있다. 또한, 클러스터 내에서 `테인트 = security`와 `톨러레이션 = intrusion`이라고 비유할 수 있다.

<aside>
💡 테인트와 톨러레이션은 클러스터 내에서 어떤 파드(벌레)가 특정 노드(사람)에 스케줄링 될 수 있도록 제한하는 역할을 한다.
</aside>

---

# 노드와 파드에서의 테인트와 톨러레이션

![fig01](/assets/img/2022-08-14-K8S_Taints and Tolerations/fig01.png)
*테인트와 톨러레이션의 예*

위 그림과 같이 노드01에 테인트가 적용되어 있다면, 해당 테인트에 톨러레이션이 있는 파드D가 아니고서야 노드 01에 스케줄링 될 수 없다.

![fig02](/assets/img/2022-08-14-K8S_Taints and Tolerations/fig02.png)
*테인트와 톨러레이션을 적용한 후, 스케줄링된 결과의 예시1*

특히 주의해야 할 점은, 파드 D가 반드시 톨러레이션이 있다고 해서 반드시 해당 노드에 배치되어야만 하는 것은 아니다. 즉, 아래와 같은 배치가 가능하다는 말이다.

![fig03](/assets/img/2022-08-14-K8S_Taints and Tolerations/fig03.png)
*테인트와 톨러레이션을 적용한 후, 스케줄링된 결과의 예시2*

<aside>
💡 테인트와 톨러레이션은 특정 파드가 반드시 특정 노드에 위치해야만 한다고 지정하는 것은 아니다.
</aside>


## 명령어로 설정하는 테인트

`kubectl taint nodes <노드의 이름> <키 값>=<밸류 값>:<테인트 효과>` 명령어를 통해 설정할 수 있다.

**테인트효과**는 아래의 값을 가질 수 있다.
  - `NoSchedule` : 톨러레이션 설정이 없으면 파드를 스케줄링 하지 않는다. 기존에 실행되던 파드에는 적용되지 않는다.
  - `PreferNoSchedule` : 톨러레이션 설정이 없으면 파드를 스케줄링하지 않는다. 하지만 클러스터 안 자원이 부족하면 테인트를 설정한 노드에서도 파드를 스케줄링 할 수 있다. 즉, 해당 노드에 톨러레이션이 없는 파드가 항상 스케줄링되는 것을 보장하지는 않는다.
  - `NoExecute` : 톨러레이션 설정이 없으면 파드를 스케줄링하지 않는다. 특히, 기존 파드도 톨러레이션 설정이 없으면 종료시킨다.

```bash
$ kubectl taint nodes node01 app=grey:NoScedule
```

이때, `app=grey`이 일치하는 톨러레이션이 없으면 파드를 node01에 스케줄링 할 수 없다.

또한 테인트 적용 취소 시는 ‘-’ 명령을 추가한다.

```bash
$ kubectl taint nodes node01 app=grey:NoScedule-
```


## 파드 정의 YAML로 설정하는 톨러레이션

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations: # 톨러레이션이 정의되는 부분
  - key: "app" # 테인트에서의 key 값
    operator: "Equal" # 테인트에서의 "="에 대응됨
    value: "grey" # 테인트에서의 value 값
    effect: "NoSchedule" # 테인트 효과
```

---

# 마스터 노드에 파드가 배치되지 않는 이유

마스터 노드에 파드가 배치되지 않는 이유도 사실은 테인트 때문이다. 마스터 노드는 쿠버네티스가 처음 설치될 때 자동적으로 테인트가 적용된다.
`kubectl describe node kubemaster | grep Taint` 명령을 치면, 마스터 노드에 테인트가 적용되어 있는 것을 확인할 수 있다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `김징어의 Devlog` : [[k8s] 노드 스케쥴링 - Taints와 Toleratioin(테인트와 톨러레이션)](https://kimjingo.tistory.com/146)
