---
title: (K8S) 노드 컴포넌트
author: simon sanghyeon
date: 2022-08-03
categories: [Kubernetes]
tags: [Kubernetes, k8s, architecture]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

# Kubelet이란?
쿠버네티스의 각 노드를 배로 비유한다면, `kubelet`은 배의 선장이라고 비유할 수 있다. kubelet은 클러스터의 각 노드에서 실행되는 에이전트로서 노드에서 컨테이너가 동작하도록 관리하는 핵심 요소라고 할 수 있다.
각 노드에서 컨트롤 플레인에 있는 kube-apiserver와 통신하며, 요청이 있을 때 노드에서 파드를 생성 혹은 변경하는 역할을 담당한다.

예를 들어, 파드 정의 YAML 파일은 kubectl 명령어를 통해서 적용되는데, 이 YAML의 내용이 kube-apiserver를 통해 kubelet에 전달되며, kubelet이 YAML의 내용대로 파드를 관리한다.
참고로, kubelet은 쿠버네티스를 통해서 생성된 컨테이너가 아니라면 관리하지 않는다.

# Kube-proxy란?
쿠버네티스 클러스터 내부에는 별도의 가상 네트워크들이 존재한다. `kube-proxy`는 이러한 가상 네트워크가 동작할 수 있도록 하는 네트워크 프록시로, 쿠버네티스의 service 개념의 구현부이다.

쿠버네티스에서 네트워크는 매우 복잡하지만, 간단하게 살펴보자. 쿠버네티스 내부에 위치한 특정 파드로 요청을 보내기 위해서는 해당 파드의 IP를 정확히 알아야 하지만, 쿠버네티스에서 파드의 IP는 배포될 때마다 매번 변경된다.
파드의 IP가 매번 바뀌더라도 정확하게 특정 파드로 요청이 전달되도록 관리하는 컴포넌트가 바로 kube-proxy다.

# Container Runtime이란?
`container runtime`은 실제로 컨테이너를 실행시키는 역할을 수행하는 애플리케이션을 의미한다. 가장 유명한 container runtime은 `도커(docker)`가 있으며, 이외에도 `컨테이너디(containerd)`, `크라이오(CRI-O)` 등이 존재한다.
쿠버네티스는 컨테이너를 관리하기 위해서 특정 애플리케이션을 사용할 것을 강제하지는 않으며, 단지 쿠버네티스가 컨테이너를 제어하기 위해 제공하는 표준 규약인 `Container Runtime Interface(CRI)`를 준수할 것을 요구한다.

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `Samsung SDS 블로그` : [쿠버네티스 알아보기 3편: 쿠버네티스를 이루고 있는 여러 가지 구성 요소](https://www.samsungsds.com/kr/insights/kubernetes-3.html)<br>
[4] `쿠버네티스 공식 documentation` : [쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)
