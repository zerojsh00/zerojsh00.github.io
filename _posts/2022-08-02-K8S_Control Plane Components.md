---
title: (K8S) 컨트롤 플레인 컴포넌트
author: simon sanghyeon
date: 2022-08-02
categories: [Kubernetes]
tags: [Kubernetes, K8S, Architecture]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

# Kube-API Server란?
`kube-apiserver`는 쿠버네티스 API를 노출하는 컴포넌트로, 쿠버네티스에서 가장 중요한 역할을 담당한다.
kube-apiserver는 클러스터로 요청이 왔을 때 해당 요청이 유효한지 검증하고, RESTful API를 이용하여 적절한 컴포넌트에서 요청을 처리할 수 있도록 전달하는 등 모든 과정의 중심 역할을 한다.

kube-api server는 kubectl 명령어가 들어왔을 때 다음을 관장한다.
  - Authenticate User
      - 유저에 대한 인증
  - Validate Request
      - 요청에 대한 유효성 검증
  - Retrieve Data
      - ETCD로부터 요청에 대한 데이터를 가져옴
  - Update ETCD
      - 데이터 변경 건에 대해서 ETCD를 업데이트 시켜줌
  - Scheduler와의 통신
      - scheduler가 어떤 worker node에서 요청을 처리할지(예를 들어, 어디에 파드를 생성할지 등)를 결정하면 kube-apiserver에게 그 정보를 전송함
  - Kubelet과의 통신
      - kube-api server는 적합한 worker node에 있는 kubelet에게 요청 처리에 대한 정보를 전송하면, kubelet이 요청을 해당 노드에서 처리함

# ETCD란?
`ETCD(엣시디)`는 분산형 key-value 저장소로, 쿠버네티스의 기본 데이터 저장소다.
모든 쿠버네티스 클러스터 상태를 저장하는 일종의 데이터베이스인 것이다.
예를 들어, ETCD 데이터 저장소는 노드, 파드, configs, secrets, accounts, roles, bindings 등 클러스터에 관한 전반적인 정보를 저장한다.
따라서 쿠버네티스에서 어떤 노드를 추가한다거나, 파드 또는 레플리카셋을 배포하는 등 클러스터에서 발생하는 모든 변화들에 따라 ETCD의 상태가 변화된다.

ETCD에 저장된 데이터는 반드시 `kube-apiserver`를 통해서만 접근할 수 있다.
예를 들어, `kubectl get pods`와 같은 명령어를 실행하면, kube-apiserver에 요청이 전달되고, kube-apiserver가 ETCD 데이터를 읽어와 kubectl 사용자에게 결과를 반환하게 된다.

## ETCD 설치 : 클러스터를 처음부터 구축하는 경우
ETCD binaries를 **master node**에 직접 다운로드 받고, binaries를 설치하고 설정을 하는 등 밑바닥부터 ETCD를 구축해야 한다.
etcd.service를 살펴보면 `--advertise-client-urls https://${INTERNAL_IP}:2379$\\` 라인이 있는데, 2379는 ETCD의 default port이다. 여기서의 URL은 kube-apiserver에 설정되어야 하는데, 그래야만 클러스터의 오케스트레이션을 최종적으로 담당하는 kube-apiserver가 ETCD 서버에 접근할 수 있다.

## ETCD 설치 : 클러스터를 kubeadm tool을 통해서 구축하는 경우
kubeadm으로 클러스터를 구축하는 경우, kubeadm이 알아서 ETCD 서버를 파드(pod)의 형태로 `kube-system 네임스페이스`에 배포해준다. 이 파드 안에 있는 etcdctl utility를 이용하면 ETCD 데이터베이스를 직접 둘러볼수도 있다.
```bash
kubectl get pods -n kube-system

NAMESPACE   NAME        READY STATUS  RESTART AGE
... 생략 ...
kube-system etcd-master 1/1   Running 0       1h
... 생략 ...
```

고가용 환경에서는 master node 자체가 여러 개일 것이다. 이때는 ETCD 인스턴스가 여러 master node에 골고루 존재할 것이다. 이 경우, ETCD service configuration을 통해서 적합한 파라미터를 세팅함으로써 ETCD 인스턴스가 서로를 알 수 있도록 해야한다.
예를 들어, `--initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller-1=https://${CONTROLLER1_IP}:2380`와 같이 설정해야 한다.

# Kube-Controller-Manager란?
`kube-controller-manager`는 쿠버네티스에 있는 다양한 `컨트롤러(controller)`들을 관리하는 컴포넌트다.

컨트롤러는 클러스터의 상태를 지속적으로 관찰하며 현재의 상태를 원하는(desired) 상태로 변경시키는 역할을 한다.
대표적으로 `노드 컨트롤러`, `레플리케이션 컨트롤러`가 있다.
예를 들어, 노드 컨트롤러의 경우, kube-apiserver를 통해서 매 5초마다 노드를 모니터링하며, 애플리케이션이 계속 실행될 수 있도록 조치를 취한다. 즉, 노드로부터 마치 심장박동(heartbeat)같은 상태 정보를 받는데, 상태 정보가 끊기면 UNREACHABLE 표기를 한다. UNREACHABLE 상태가 수 초간 유지되면 해당 노드의 파드를 제거하고 건강한 노드에 provision 한다.
레플리케이션 컨트롤러의 경우, replicaset을 모니터링하여 원하는 수량 만큼의 파드가 잘 실행되고 있는지를 보장한다. 만약 파드가 죽으면 복구해낸다.
이외에도 컨트롤러는 `엔드포인트 컨트롤러`, `서비스 어카운트 & 토큰 컨트롤러` 등 다양하게 존재한다.

kube-controller-manager는 위와 같은 컨트롤러들을 실행하고 관리한다.

# Kube Scheduler란?
쿠번네티스 클러스터는 여러 노드로 구성되어 있다. `kube-scheduler`는 새로운 파드들이 만들어질 때, 현재 클러스터 내에서 자원 할당이 가능한 최적의 노드에 파드가 배포되도록 스케줄링하는 역할을 수행한다.
kube-scheduler는 스케줄링을 위해 특정한 CPU나 메모리 리소스에 대한 요구사항이라든지 각종 제약사항을 종합적으로 판단한다.

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `아리수 님의 블로그` : [쿠버네티스(kubernetes) 구성요소](https://arisu1000.tistory.com/27828)<br>
[4] `Samsung SDS 블로그` : [쿠버네티스 알아보기 3편: 쿠버네티스를 이루고 있는 여러 가지 구성 요소](https://www.samsungsds.com/kr/insights/kubernetes-3.html)<br>
[5] `쿠버네티스 공식 documentation` : [쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)
