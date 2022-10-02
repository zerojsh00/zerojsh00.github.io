---
title: (K8S) 퍼시스턴트 볼륨(Persistent Volume)
author: simon sanghyeon
date: 2022-09-05
categories: [Kubernetes]
tags: [Kubernetes, K8S, Storage, Volume, Persistent Volume]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# YAML을 이용한 볼륨 마운트의 문제
앞서 [쿠버네티스에서 파드 정의 YAML 파일을 이용해서 볼륨의 정보를 직접 입력해 사용하는 방법](https://zerojsh00.github.io/posts/Volumes/)을 살펴보았다.
볼륨을 간단하게 테스트하기 위한 용도라면 이와 같은 방법으로 사용해도 크게 문제될 것은 없다.
그렇지만 실제로 애플리케이션을 개발한 뒤 YAML 파일로 배포할 때는 이러한 방법이 바람직하지 않을 수 있다.

파드를 아주 많이 사용하는 환경을 고려하면, 파드 정의 YAML 파일마다 일일이 어떤 볼륨을 마운트 할지를 기록해야 한다.
**만약 volume mount 관련한 변경 사항이 있다면, 모든 YAML 파일을 수정해야만 한다.**

또한 YAML 파일에 특정 네트워크 스토리지(NFS, Gluster FS, iSCSI, AWS EBS …)를 사용해야 한다고 명시해두었다고 할 때, 만약 이 YAML 파일을 다른 개발 부서에 배포하거나 웹상에 공개해야 할 경우에도 문제가 될 수 있다.
예를 들어, YAML 파일에 네트워크 볼륨으로 NFS를 사용해야 한다고 명시해서 고정해두었다면, 다른 네트워크 볼륨은 사용할 수 없고 반드시 NFS를 사용해야만 하는 문제가 있다.
NFS가 아닌 Gluster FS나 iSCSI 등을 사용하고 싶다면, 해당 네트워크 볼륨 타입을 명시하는 별도의 YAML 파일을 여러 개 만들어 배포해야 한다.

---
# 퍼시스턴트 볼륨(PV)과 퍼시스턴트 볼륨 클레임(PVC)
이러한 불편함을 해결하기 위해서 쿠버네티스에서는 `퍼시스턴트 볼륨(Persistent Volume, PV)`과 `퍼시스턴트 볼륨 클레임(Persistent Volume Claim, PVC)`이라는 오브젝트를 제공한다.
 **핵심 아이디어는 파드 정의 YAML에서 네트워크 볼륨이 NFS인지, Gluster FS인지, AWS EBS인지 등을 명시하지 않더라도 적합한 볼륨을 사용할 수 있도록 하자는 것이다.**

![fig01](/assets/img/2022-09-05-Persistent Volumes/fig01.png)

우선, 쿠버네티스 클러스터를 관리하는 `인프라 관리자`와 애플리케이션을 배포하려는 `사용자(개발자)`가 나뉘어 있다고 하자.
**`인프라 관리자`는 NFS, Ceph와 같은 여러 종류의 네트워크 볼륨을 쿠버네티스로 가져와서 PV 리소스를 미리 생성해둔다.**
즉, 여러가지 네트워크 볼륨(스토리지) 서버에 접근할 수 있는 엔드 포인트를 준비해두는 것이다.

이후, **`사용자(개발자)`는 YAML 파일에 “이 파드는 데이터를 저장해야 하므로, 마운트 할 수 있는 외부 볼륨이 필요하다”는 의미로 PVC를 명시한다.**
그러면 쿠버네티스는 인프라 관리자가 생성한 PV의 속성과 사용자가 요청한 PVC 요구 사항이 일치한다면, 두 개의 리소스를 매칭시켜 바인드 한다.

# 퍼시스턴트 볼륨 생성하기
```yaml
# pv-definition.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1 # PV의 이름
spec:
  accessModes:
    - ReadWriteOnce # ReadOnlyMany | ReadWriteOnce | ReadWriteMany
  capacity:
    storage: 1Gi # PV로 사용하는 스토리지의 용량
  hostPath: # 스토리지 역할을 할 호스트의 폴더 경로, not recommended
    path: /tmp/data
```

퍼시스턴트 볼륨은 위와 같이 PersistentVolume 정의 YAML 파일을 작성한 후, `kubectl create -f pv-definition.yaml` 명령을 통해 생성할 수 있으며, `kubectl get persistentvolume` 명령을 통해 확인할 수 있다.

단, 위와 같이 스토리지 역할을 할 호스트의 폴더 경로를 직접 지정하는 방식은 여러 개의 노드를 사용할 경우, 쿠버네티스가 스토리지의 경로 및 그 안에 있을 데이터가 모두 동일하다고 간주할 수 있으므로 문제의 여지가 있다.

**따라서 아래의 예와 같이 여러 네트워크 스토리지들 중 하나를 사용하도록 설정하는 것이 바람직하다.**

```yaml
# pv-definition.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1 # PV의 이름
spec:
  accessModes:
    - ReadWriteOnce # ReadOnlyMany | ReadWriteOnce | ReadWriteMany
  capacity:
    storage: 1Gi # PV로 사용하는 스토리지의 용량
  awsElasticBlockStore: # AWS EBS를 사용하는 경우 다음과 같이 설정한다.
    volumeID: <volume-id>
    fsType: ext4
```
---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
