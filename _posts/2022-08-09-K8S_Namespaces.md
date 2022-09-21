---
title: (K8S) Namespaces
author: simon sanghyeon
date: 2022-08-09
categories: [Kubernetes]
tags: [Kubernetes, K8S, Namespace]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

# 네임스페이스(Namespacce) : 리소스를 논리적으로 구분하는 장벽
쿠버네티스에서는 리소스를 논리적으로 구분하기 위해 `네임스페이스(Namespace)`라는 오브젝트를 제공한다.
간단히 생각해서 **네임스페이스는 파드, 레플리카셋, 디플로이먼트, 서비스 등과 같은 쿠버네티스 리소스들이 묶여 있는 하나의 가상 공간 또는 그룹**이라고 이해할 수 있다.

예를 들어, 모니터링을 위한 모든 리소스들은 monitoring이라는 이름의 네임스페이스에서 생성할 수 있고, 테스트를 위한 리소스들은 tested라는 네임스페이스에서 생성할 수 있다. 또는 여러 개발 조직이 하나의 쿠버네티스 클러스터를 공유해 사용해야 한다면, 조직별로 네임스페이스를 사용하도록 구성할 수도 있다.
이처럼 여러 개의 네임스페이스를 사용하면 마치 하나의 클러스터에서 여러 개의 가상 클러스터를 동시에 사용하는 것처럼 할 수 있다.

---

# 기본적인 네임스페이스
네임스페이스는 직접 생성하지 않았더라도 기본적으로 `default` 네임스페이스, `kube-public` 네임스페이스, `kube-system` 네임스페이스가 있다.

## default 네임스페이스
클러스터가 처음 구축되면, 자동으로 default 네임스페이스가 생성된다.
`kubectl` 명령어는 기본적으로 별도로 `--namespace` 옵션을 명시하지 않으면 default 네임스페이스를 사용한다.

## kube-system 네임스페이스
`kube-system`는 DNS service와 같은 네트워킹 솔루션 등 일반 사용자들에 의해 함부로 변경되면 안 되는 필수 컴포넌트들과 설정 값들이 위치한 네임스페이스이다.
kube-system 네임스페이스는 쿠버네티스에 대한 충분한 이해 없이는 건드리지 않는 것이 좋다.

## kube-public 네임스페이스
모든 사용자로부터 사용 가능한 리소스가 만들어지는 곳이다.

---

# 네임스페이스가 왜 필요할까?
작은 규모의 쿠버네티스 운영 환경에서는 default 네임스페이스만으로도 운영할 수 있을 것이다. 그런데 쿠버네티스 클러스터가 점차 커져서 production 목적으로 이용하게 되면, 네임스페이스 관리가 필요해진다.

예를 들어, 개발 목적의 Dev 네임스페이스와 제품레벨에서 배포 목적의 Prod 네임스페이스를 분리해서 사용할 수 있다. 이러한 방법으로 개발단에서 실수로 배포 네임스페이스 상의 리소스를 변경하는 일을 차단할 수 있다.

또한 각각의 네임스페이스에서는 각각 적용할 수 있는 규칙을 정할 수 있다. 예컨대, 리소스 사용에 대한 한계, 즉 resource limits도 네임스페이스에 따라 설정할 수 있다.
예를 들어 `Resource Quata`라고 하는 오브젝트를 이용해서 특정 네임스페이스에서 생성되는 파드의 자원 사용량을 제한할 수 있다.

---
# 네임스페이스 사용 방법

## 파드 생성 시 네임스페이스를 지정하는 방법
`kubectl create -f pod-definition.yml --namespace=<네임스페이스 이름>`으로 지정하거나, 혹은 파드 정의 YAML 파일의 metadata 섹션에서 `namespace: <네임스페이스 이름>`으로 지정할 수 있다.

## 직접 네임스페이스를 만드는 방법
```yaml
# namespace-dev.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
네임스페이스 정의 YAML 파일을 생성한 후 `kubectl create -f namespace-dev.yaml` 명령으로 생성할 수 있다.

## 네임스페이스 설정 방법
`kubectl config set-context --current --namespace=<네임스페이스 이름>` 명령으로 네임스페이스를 설정한다.

## 네임스페이스에서 Resource Quota를 통해 파드 자원 사용량을 제한하는 방법

```yaml
# compute-quota.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev

spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
`kubectl create -f compute-quota.yaml` 명령으로 적용한다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
