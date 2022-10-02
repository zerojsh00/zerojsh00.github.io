---
title: (K8S) 퍼시스턴트 볼륨 클레임(Persistent Volume Claim)
author: simon sanghyeon
date: 2022-09-06
categories: [Kubernetes]
tags: [Kubernetes, K8S, Storage, Volume, Persistent Volume Claim]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# Recap : 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임
앞서 [정리한 글](https://zerojsh00.github.io/posts/Persistent-Volumes/)에서 퍼시스턴트 볼륨(PV)과 퍼시스턴트 볼륨 클레임(PVC)에 대한 개념을 살펴보았다.
퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임은 각각 별도의 오브젝트로서, `퍼시스턴트 볼륨`은 쿠버네티스 `클러스터 인프라 관리자`가 미리 생성해두는 것이고, `퍼시스턴트 볼륨 클레임`은 `사용자(개발자)`가 스토리지를 이용하기 위해서 생성하는 것이다.

쿠버네티스는 퍼시스턴트 볼륨 클레임을 `sufficient capacity`, `access modes`, `volume modes`, `storage class`와 같은 요청 속성을 고려하고, 이에 적합한 퍼시스턴트 볼륨과 매칭하여 바인딩 한다.

이때 모든 PVC는 단 한 개의 PV과 바인딩 된다.
그러나, 이러한 요청 속성을 고려했음에도 하나의 PVC에 여러 개의 PV가 적합하다고 매칭될 경우, `labels`와 `selector`를 이용해서 특정 레이블과 매칭되는 조합으로 바인딩하는 것이 가능하다.

![fig01](/assets/img/2022-09-06-Persistent Volume Claims/fig01.png){: width="500" height="500"}

만약, PVC에 매칭 될 수 있는 PV가 존재하지 않는다면, PVC는 새로운 볼륨이 생겨나기 전까지 pending 상태로 머물러있게 된다. 이후, 새로운 볼륨이 생겨나면 자동으로 pending 상태의 PVC와 바인딩 된다.

---
# 퍼시스턴트 볼륨 클레임 생성하기

![fig02](/assets/img/2022-09-06-Persistent Volume Claims/fig02.png)

PV와 유사하게, PVC를 생성하는 방식은 위와 같다. 마찬가지로 `kubectl get persistentvolumeclaim` 명령을 통해서 생성된 PVC를 확인할 수 있다.

---
# 퍼시스턴트 볼륨 클레임 삭제하기

`kubectl delete persistentvolumeclaim {PVC 이름}` 명령을 통해서 생성했던 PVC를 삭제하는 것 또한 가능하다.

---
# 퍼시스턴트 볼륨 반환 정책

PV와 바인딩된 PVC를 삭제하면, PV는 어떻게 될까? PV정의 YAML 파일에서 PV가 어떻게 처리될지 직접 설정할 수 있는데, 이를 `PV 반환 정책`이라고 한다.

## Retain 설정
YAML 파일 내의`persistentVolumeReclaimPolicy` 설정에서 정의할 수 있는데, default는 `Retain`으로, PVC를 삭제하여 PV가 released 상태가 되더라도 관리자가 직접 PV를 제거하지 않는 한 PV는 남아있게 하겠다는 설정이다. PV가 남아있다 하더라도 PV에 아직 데이터가 남아있기(retain) 때문에, 다른 PVC에 의해 사용될 수 있는 상태는 아니다.

## Delete 설정
`Delete`로 설정할 경우, PVC가 삭제되어 PV가 released 상태가 되면 PV와 연결된 디스크 내부 자체가 자동으로 함께 제거된다.

## Recycle 설정
`Recycle`로 설정할 수 있는데 deperecated 되어있다.

---
# 파드에서 퍼시스턴트 볼륨 클레임 사용하기
PVC 생성을 완료했으면, 아래의 예제와 같이 파드 정의 YAML 파일 내 `persistentVolumeClaim` 설정 부분에서 PVC 이름을 기입하여 사용할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
       - mountPath: "/var/www/html"
         name: mypd
   volumes:
     - name: mypd
       persistentVolumeClaim:
         claimName: myclaim
```

레플리카셋이나 디플로이먼트에서도 동일한 방식으로 사용할 수 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
