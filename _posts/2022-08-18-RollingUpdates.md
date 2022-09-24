---
title: (K8S) 롤링 업데이트(Rolling Update)
author: simon sanghyeon
date: 2022-08-18
categories: [Kubernetes]
tags: [Kubernetes, K8S, RollingUpdate]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 디플로이먼트 롤아웃과 리비전
앞서, [디플로이먼트](https://zerojsh00.github.io/posts/K8S_Deployment/)는 레플리카셋의 변경 사항을 저장하는 `리비전(revision)`을 남기며, 문제가 발생할 경우 이를 이용하여 `롤백(rollback)`을 할 수 있다는 장점을 살펴보았다.
이 과정을 그림으로 간단하게 예를 들면 아래와 같다.

![fig01](/assets/img/2022-08-18-RollingUpdates/fig01.png)

최초에 디플로이먼트를 생성하면, `롤아웃(rollout)`이 시작되는데, 이 롤아웃은 위 그림과 같이 nginx:1.7.0 버전들로 이루어진 디플로이먼트 `revision 1`을 만들어낸다고 하자.
또한 시간이 지나서 애플리케이션(즉, 컨테이너)이 업그레이드 될 때도 롤아웃이 진행되는데, 이 롤아웃은 nginx:1.7.1 버전들로 이루어진 디플로이먼트 `revision 2`를 만들어낸다고 하자.

이처럼 리비전을 통해 변화들이 추적되기 때문에, 필요에 따라 이전 디플로이먼트 버전으로 롤백 할 수 있다.

## 롤아웃 명령어(Rollout Command)
- `kubectl rollout status deployment/<deployment 이름>` 명령어를 통해서 해당 deployment의 rollout 상태를 확인할 수 있다.
- `kubectl rollout history deployment/<deployment 이름>` 명령어를 통해서 해당 deployment의 revision history 정보를 알 수 있다.

---
# 디플로이먼트 전략(Deployment Strategy)

![fig02](/assets/img/2022-08-18-RollingUpdates/fig02.png)

디플로이먼트의 업데이트 전략은 두 가지, 즉, `롤링 업데이트(Rolling Update)`와 `재생성(Recreate)`가 있다.
`.spec.strategy.type`에서 전략을 명시할 수 있으며, 디플로이먼트의 특별한 배포 전략을 설정하지 않으면, 롤링 업데이트가 기본값이다.

## 재생성(Recreate)
`재생성(Recreate)` 전략은 애플리케이션의 모든 이전 버전의 인스턴스들을 한번에 다운시킨 후 한꺼번에 업데이트를 진행하는 방식이다.
즉, 업데이트 전에 파드 종료를 보장할 수 있다. 그러나 이전 버전의 모든 인스턴스들이 다운되었을 때 애플리케이션이 다운되어 사용자들이 접근할 수 없게 되는 문제가 있다.

## 롤링 업데이트(Rolling Update)
`롤링 업데이트(Rolling Update)`는 이전 버전의 인스턴스를 하나씩 번갈아가면서 업데이트 하는 방식이다. 롤링 업데이트 전략은 모든 인스턴스가 동시에 다운되지 않기 때문에 애플리케이션이 다운되는 문제를 해결할 수 있다.

---
# 롤링 업데이트 및 롤백 방법
## 롤링 업데이트
롤링 업데이트의 가장 쉬운 방법은 디플로이먼트 정의 YAML 파일 내 상세 정보를 원하는 버전에 맞게 변경한 후, `kubectl apply -f <deployment 이름>.yaml` 명령어를 입력하여 수행할 수 있다.

YAML 파일을 이용하지 않고서도 변경할 수는 있다. 예를 들어서, 디플로이먼트 이름을 myapp-deployment라고 한다면, `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`와 같이 `kubectl set` 명령어를 사용하고 `nginx:1.9.1`과 같이 직접 특정 버전을 기입함으로써 원하는 버전으로 명령어만을 이용하여 롤링 업데이트 할 수 있다.
다만, 이 경우에는 디플로이먼트 정의 YAML 파일의 내용은 변경되지 않으므로, 미래에 해당 YAML을 다시 사용할 경우 버전 착오에 유의해야 한다.

## 롤백
롤링 업데이트 이전 상태로 되돌리고 싶다면, kubectl rollout undo deployment/<deployment 이름> 명령어를 실행하여 rollback 할 수 있다.

---
# 롤링 업데이트가 진행되는 모습의 예
![fig03](/assets/img/2022-08-18-RollingUpdates/fig03.png){: width="500" height="500"}

디플로이먼트는 롤링 업데이트를 위해서 레플리카셋을 하나 더 생성하고, 기존 레플리카셋 내에 있는 파드를 하나씩 삭제함과 동시에 새로운 레플리카셋 내에서 파드를 하나씩 업데이트 하는 방식으로 롤링 업데이트를 진행한다.

![fig04](/assets/img/2022-08-18-RollingUpdates/fig04.png){: width="500" height="500"}

위와 같이 `kubectl get replicasets`명령어를 통해서 두 개의 레플리카셋이 활용되는 모습을 확인할 수 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
