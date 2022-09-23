---
title: (K8S) 파드에 필요한 리소스 설정하기
author: simon sanghyeon
date: 2022-08-17
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling, Resource]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

쿠버네티스의 스케줄러(scheduler)는 어떤 파드를 어떤 노드에 배치할지 결정한다. 즉, 해당 파드가 필요로하는 자원의 정도를 살펴보고, 이를 지원할 수 있는 적합한 노드에 파드를 배치한다.
가용 자원으로 파드를 실행할 수 없는 상태인 노드는 피하고, 가용 자원이 적당한 노드에 파드를 배포하는 것이다.

만약, 그 어떠한 노드도 파드를 실행할 수 없는 상태라면, 쿠버네티스는 스케줄링을 중단하고 파드 배포를 `pending`한다.
pending 상태의 event를 예를 들면, `FailedScheduling No nodes are available that match all of the following predicates:: Insufficient cpu (3).`과 같은 에러 메시지를 볼 수 있다.

이번 글은 **파드를 실행하는 데 있어서 필요한 자원을 명시적으로 설정하는 방법**을 다룬다.

# 파드 생성 시 Resource Request

다음과 같이 파드 정의 YAML 파일을 통해 파드를 구동하기 위해 필요한 리소스를 명시적으로 설정할 수 있다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    resources: # 아래 부분을 설정함으로써 파드 실행에 필요한 노드의 자원을 정할 수 있다.
      requests:
        memory: "1Gi"
        cpu: 1
      limits: # default 제한 값이 싫은 경우, 아래에 설정할 수 있다.
        memory: "2Gi" # default는 512Mi이다.
        cpu: 2 # default는 1(vCPU)이다.
```

**컨테이너 자체적으로는 노드에서 얼마나 많은 리소스를 사용할지 결정할 수 없다.**
그래서 자신이 써도 되는 수준의 리소스보다 더욱 많이 사용하게 될 수 있고, 이는 결국 다른 컨테이너나 노드에 영향을 주게 된다.
따라서 **파드에서 컨테이너에 대한 자원을 설정해서 제한하는 방법을 사용해야 한다.**

별다른 설정을 명시적으로 하지 않으면, 쿠버네티스는 컨테이너들에 default로 자원을 제한하여 할당하는데, CPU의 경우는 1 vCPU 만큼, 메모리의 경우 512Mi 만큼을 할당한다.
default 제한 값이 마음에 들지 않을 경우, `limits`를 명시적으로 기재하여 설정할 수 있다.

클러스터가 제한된 CPU 값을 초과하려 하면, 쿠버네티스가 이를 `throttle`하여 막는다. 즉, 제한값을 초과하지 못하는 것이다. 반면에 메모리 사용의 경우는 컨테이너의 자원 사용이 제한된 값을 넘을 수는 있는데, 이때는 파드가 `terminate` 되고 만다.

---

# 리소스의 단위
## CPU 단위
CPU의 단위는 `m`으로도 사용할 수 있는데, 이때 m은 `milli`, 즉, CPU 1개의 1/1000을 의미한다. 예를 들어, CPU 0.1은 100m으로 표기될 수 있다. 최소 1m까지 값을 가질 수 있다.

1 CPU는 1 `vCPU(가상 CPU 코어)`와 동일하다. 이는 `1 AWS vCPU`, `1 GCP Core`, `1 Azure Core` 등과 동일한 수준이다.

## 메모리 단위
메모리의 단위는 보통 `Mi`를 사용하는데, 이는 `MiB(메비바이트)`를 의미한다. 참고로, 엄밀하게는 MiB와 MB는 다르지만, 정교한 SW가 아니라면 동일한 의미로 사용해도 무방하다.

- 참고
  - 1 G(Gigabyte) = 1,000,000,000 bytes
  - 1 M(Megabyte) = 1,000,000 bytes
  - 1 K(Kilobyte) = 1,000 bytes

  - 1 Gi(Gibibyte) = 1,073,741,824 bytes
  - 1 Mi(Mebibyte) = 1,048,576 bytes
  - 1 Ki(Kibibyte) = 1,024 bytes


---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
