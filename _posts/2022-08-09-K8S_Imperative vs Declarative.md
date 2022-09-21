---
title: (K8S) 명령형과 선언형
author: simon sanghyeon
date: 2022-08-09
categories: [Kubernetes]
tags: [Kubernetes, K8S, Imperative, Declarative]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

# 명령형(Imperative)과 선언형(Declarative)
## 명령형 (specify what to do and how to do)
도커는 컨테이너를 다룰 때 `docker run` 명령과 같이 특정 명령을 처리하는 주체와 통신해서 작업을 수행한 후, 작업에 대한 결과를 반환 받는 방식으로 작동한다.
이를 `명령형(Imperative)`라고 부른다. 물론 쿠버네티스에서도 `kubectl run --image=nginx nginx`와 같이 명령형 방식으로 작동할 수 있다.

## 선언형 (specify what to do, not how to do)
반면, 쿠버네티스는 최종적으로 만들고자 하는 `바람직한 상태(Desired State)`를 YAML로 정의한 후 `kubectl apply -f {YAML파일}` 명령으로 실행할 수도 있다.
이처럼 최종적으로 달성해야 하는 상태를 정의하여 실행하는 방식을 `선언형(Declarative)`이라고 부른다.

---

# 명령형과 선언형의 예
## 명령형의 예
**Create Objects**

- `kubectl run --image=nginx nginx`
- `kubectl create deployment --image=nginx nginx`
- `kubectl expose deployment nginx --port 80`

YAML 없이도 가능하지만, 복잡한 기능을 추가할 수 없으며, 복잡한 환경에서 활용하기 어렵다.

**Update Obejcts**

- `kubectl edit deployment nginx`
- `kubectl scale deployment nginx --replicas=5`
- `kubectl set image deployment nginx nginx=nginx:1.18`

- `kubectl create -f nginx.yaml`
- `kubectl replace -f nginx.yaml`
- `kubectl delete -f nginx.yaml`

명령형 방식을 사용한다면, 관리자가 반드시 현재의 모든 configuration을 알고 있어야 하며, 컴포넌트들에 변경을 주고 싶다면, 그전에 에러가 나지 않도록 모든 조치를 취해야 한다. 예를 들어, 기존에 있던 파드를 또 생성한다고 하면 AlreadyExists 에러가 난다.

## 선언형의 예
YAML을 미리 생성한 후, `kubectl apply -f nginx.yaml`를 실행한다. configuration에 변경이 있어도 에러가 나지 않는다.

---

# 참고 : CKA 시험에서의 팁
명령형 방식이 시간 단축에는 효과적일 것이다. 반면, 복잡한 문제는 multiple containers, 환경변수, init container 등등 configuration을 많이 건드려야 할텐데, 그땐 선언형 방식이 나을 것이다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
