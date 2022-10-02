---
title: (K8S) 스토리지 클래스와 다이나믹 프로비저닝
author: simon sanghyeon
date: 2022-09-07
categories: [Kubernetes]
tags: [Kubernetes, K8S, Storage, Volume, Storage Class, Provisioning]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 182강 Storage Class

앞서 [PV](https://zerojsh00.github.io/posts/Persistent-Volumes/)와 [PVC](https://zerojsh00.github.io/posts/Persistent-Volume-Claims/)를 생성하는 방법을 살펴보았고, 파드 정의 YAML 파일에서 어떻게 PVC를 적용하는지도 살펴보았다.

# Static Provisioning Volume

지금까지 살펴보았던 방식은 다음과 같은 workflow를 가졌다.

- Create a **[PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)** that has volume properties like capacity, permissions, and class.
- Create a **[PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)** to request storage and bind to a persistent volume.
- Configure a **[Pod to use the volume claim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)** to mount a volume in a container.

이러한 방식을 `static provisioning` 방식이라고 한다.

static provisioning 방식을 다시 살펴보기 위해, Google Cloud persistent disk로 PVC를 생성해본다고 하자. static provisioning 방식의 문제는 PV를 생성하기 전에, 반드시 아래와 같이 Google Cloud에 디스크를 생성해야만 한다는 점이다.

```bash
gcloud beta compute disks create\
    --size 1GB
    --region us-east1
    pd-disk
```

애플리케이션이 스토리지를 필요로 할 때마다 직접 Google Cloud에서 디스크를 프로비전해야 하며, 생성한 디스크의 이름과 동일한 이름을 이용해서 다음과 같이 PV 정의 파일을 생성해야 한다.

```yaml
# pv-definition.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 500Mi
  gcePersistentDisk:
    pdName: pd-disk # 생성한 디스크의 이름과 동일한 이름을 사용한다.
    fsType: ext4
```

하지만 매번 이렇게 볼륨 스토리지(여기서는 Google Cloud 디스크)를 직접 수동으로 생성하고, 볼륨에 대한 정보를 YAML 파일에 적는 것은 번거로운 일이다.

# Storage Class와 Dynamic Provisioning

위와 같은 방식이 아니라, 애플리케이션이 볼륨 스토리지를 필요로 할 때마다 볼륨이 자동으로 프로비전 되면 훨씬 수월할 것이다. 예를 들어, Google Cloud에 자동으로 Google Storage가 프로비전되어 PVC에 따라서 자동으로 파드에 연동까지 되는 것이 지원되면 한결 편할 것이다. 이를 위해서 쿠버네티스에는 `storage class`가 활용된다. storage class는 PV의 `dynamic provisioning`을 지원한다.

dynamic provisioning은 PVC이 요구하는 조건과 일치하는 PV가 존재하지 않는다면, 자동으로 PV과 외부 스토리지를 함께 프로비저닝하는 기능이다. 따라서 dynamic provisioning을 사용하면 위에서 살펴본 바와 같이 외부 스토리지를 직접 미리 생성해두지 않아도 된다. PVC를 생성하면 외부 스토리지가 자동으로 생성되기 때문이다.

# Dynamic Provisioning 예시

static provisioning을 위해서는 `PV 정의 YAML`, `PVC 정의 YAML`, `파드 정의 YAML` 파일들이 필요했다. 그러나 dynamic provisioning은 PV을 직접 수동으로 생성하지 않기 때문에 `PVC 정의 YAML`, `파드 정의 YAML` 파일만 필요하다. 그대신, storage class(SC)를 활용하므로 `SC 정의 YAML`이 필요하다.

## SC 정의 파일의 예

```yaml
# sc-definition.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage # PVC 정의 파일에서 strageClassName 설정에 명시하여 연동한다.
provisioner : kubernetes.io/gce-pd # Google Cloud를 사용하는 경우

parameters:
  type: # 다양한 프로비저너마다 다양한 옵션이 존재한다.
  replication-type: # 다양한 프로비저너마다 다양한 옵션이 존재한다.
```

## PVC 정의 파일의 예

```yaml
# pvc-definition.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim # 파드 정의 YAML 파일 내 persistentVolumeClaim 설정에 명시하여 연동한다.
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```

## 파드 정의 파일의 예

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
