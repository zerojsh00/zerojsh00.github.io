---
title: (K8S) Security Context
author: simon sanghyeon
date: 2022-08-29
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
도커 컨테이너를 실행할 때, `docker run --user=1000 ubuntu sleep 3600` 명령과 같이 컨테이너를 실행하는 데 필요한 사용자의 ID와 같은 security 정보들을 입력해주어야 할 때가 있다. 쿠버네티스에서도 이러한 설정들을 해야하는 경우들이 있다.

쿠버네티스에서의 security 설정은 `컨테이너 레벨`에서도 할 수 있고, `파드 레벨`에서도 할 수 있다. 파드 레벨에서 security 설정을 세팅한다면, 파드 내 모든 컨테이너에 해당 설정이 적용되는 구조다. 컨테이너 레벨과 파드 레벨 둘다 설정할 경우, 컨테이너 레벨의 설정이 파드 레벨의 설정을 덮어쓴다.

# 파드 레벨에서의 Security Context 설정

아래와 같이 spec 섹션에서 securityContext 부분을 통해 설정할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:

  # 아래 부분에서 security 관련 설정을 한다.
  securityContext:
    runAsUser: 1000

  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]

```

# 컨테이너 레벨에서의 Security Context 설정

아래와 같이 securityContext 설정을 container 설정 섹션에서 정의한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]

    # container 설정 안에서 securityContext를 정의한다.
    securityContext:
      runAsUser: 1000

```

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
