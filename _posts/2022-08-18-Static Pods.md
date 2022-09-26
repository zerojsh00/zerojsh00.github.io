---
title: (K8S) 스태틱 파드(Static Pods)
author: simon sanghyeon
date: 2022-08-18
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling, StaticPods]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 마스터 노드(컨트롤 플레인)가 없는 상황 가정
어떤 파드를 어떤 노드에 배포할지 `kube-scheduler`가 결정하면, 각각의 워커 노드에 존재하는 `kubelet`은 `kubeapi-server`로부터 자신의 노드에서 어떤 파드를 어떻게 처리해야 하는지에 대한 명령을 듣고 실행한다.

만약 마스터 노드(컨트롤 플레인)의 컴포넌트들인 kube-apiserver, kube-scheduler, ETCD cluster, controller manager 등이 존재하지 않는다면 어떻게 될까?

노드를 '배'라고 비유하면 '선장'으로 비유되는 kubelet은 독립적으로 자신이 위치한 노드를 잘 관리할 수 있을까? 이 경우, 파드는 어떻게 생성되고 관리되는 것일까?

---
# Kubelet이 스스로 직접 관리하는 스태틱 파드

`스태틱 파드(Static Pods)`는 **컨트롤 플레인 컴포넌트인 kube-apiserver의 도움 없이, 특정 노드의 `kubelet` 데몬에 의해 직접 생성 및 관리되는 파드**다.

kubelet은 쿠버네티스 클러스터를 이루는 다른 컴포넌트들 없이, 독자적으로 노드를 관리할 수 있다.
파드 정의 YAML 파일과 같은 정보가 주어지면, 스스로 파드도 생성할 수 있다.
물론 컨테이너를 실행할 수 있는 도커가 설치되어 있는 상황에서 말이다.

그런데 마스터 노드가 없다는 가정에서는 kube-apiserver가 존재하지 않으므로, 어떤 파드를 만들지 등에 대한 정보를 제공받을 수 없는 상황이다.
그러나, 사실 이렇게 kube-apiserver가 없는 상황에서도 어떤 파드를 만들지에 대한 정보를 제공해주면, kubelet 스스로 문제 없이 독자적으로 파드를 생성할 수 있다.

즉, **kubelet이 읽을 수 있는 특정 directory 경로에 파드 정의 YAML 파일을 두면, kubelet이 주기적으로 해당 경로를 체크하여 파드를 생성할 수 있다.** 심지어 파드 생성 뿐만 아니라, 파드가 죽지 않고 살아있도록 보장까지 해주며, 파드 정의 YAML 파일에 변화가 있으면 파드를 재생성 해주기도 한다. 또한 파일이 사라지면, 파드를 제거하기도 한다.

이처럼, kube-apiserver 등과 같은 다른 쿠버네티스 컴포넌트 없이 kubelet만이 독자적으로 만들어내고 관리하는 파드를 `스태틱 파드`라고 부른다.

스태틱 파드가 생성되고 나면, `docker ps` 명령을 통해 컨테이너가 정상적으로 실행되고 있다는 것을 확인할 수 있다.
그러나 `kubectl` 명령어로는 확인할 수 없다. 왜나하면 kubectl 명령을 처리하기 위해서는 kube-apiserver가 필요하기 때문이다.

## Kubelet은 파드만 생성 및 관리할 수 있다.
kubelet이 관리할 수 있는 오브젝트는 파드 뿐이고, 레플리카셋이나 디플로이먼트, 서비스 등 다른 쿠버네티스 오브젝트들은 kubelet이 읽을 수 있는 특정 directory 경로에 오브젝트 정의 YAML 파일을 두더라도 생성되지 않는다.

이들은 컨트롤 플레인의 컴포넌트들을 반드시 필요로 하여 전체적인 쿠버네티스 아키텍처가 있어야만 관리될 수 있기 때문이다.

---
# Kubelet이 읽을 수 있는 매니페스트 폴더

파드 정의 YAML 파일을 kubelet이 읽어들일 수 있는 `매니페스트(minifest) 폴더` (기본)경로인 `/etc/kubernetes/manifests`에 위치시킨다.
예를 들어, `/etc/kubernetes/manifests/static-pod-example.yaml`와 같이 배치한다.
**해당 경로에 파일을 배치해 놓으면 알아서 실행된다.**

---
# 쿠버네티스 아키텍처가 갖추어진 상황에서의 스태틱 파드
그렇다면, 쿠버네티스 아키텍처가 모두 갖춰진 상황에서도 스태틱 파드는 kube-apiserver의 명령에 의해 만들어지는 다른 파드들과 함께 생성될 수 있을까? 가능하다.

이 경우, kube-apiserver는 스태틱 파드가 kubelet에 의해서 만들어졌다는 사실도 알고 있다. `kubectl get pods` 명령을 입력하면 다른 파드들과 마찬가지로 스태틱 파드 또한 나타난다. 파드의 이름을 보면 (예)static-web-node01과 같이 마지막에 node01이라는 노드 이름 또한 자동으로 붙는다.

스태틱 파드가 생성되었을 때, kubelet은 각각의 스태틱 파드에 대하여 kube-apiserver에서 `미러 파드(mirror pod)`를 생성하려고 자동으로 시도한다.
*(참고로, 미러 파드는 kubelet의 스태틱 파드를 추적하는 kube-apiserver 내부의 오브젝트다.)*
즉, 노드에서 구동되는 스태틱 파드는 kube-apiserver에 의해서 **볼 수 있지만 제어될 수는 없는 읽기 전용 파드**인 셈이다.

---
# 스태틱 파드 사용의 예
대표적인 스태틱 파드 사용 예가 바로 **컨트롤 플레인 컴포넌트를 노드에 직접 배포할 때**이다.

컨트롤 플레인 컴포넌트 오브젝트들을 정의하는 YAML 파일만 매니페스트 폴더에 넣어두면, kubelet이 스태틱 파드를 실행하여 해당 노드를 마스터 노드로 만들 수 있다.
즉, 스태틱 파드는 컨트롤 플레인과 독립적으로 작동되므로, `controller-manager.yaml`, `apiserver.yaml`, `etcd.yaml`과 같은 파일을 지정된 메니페스트 폴더에 넣음으로써 컨트롤 플레인의 컴포넌트들을 노드에 직접 배포할 수 있는 것이다. 이들 중 어떠한 스태틱 파드에 충돌이 난다 하더라도, kubelet이 다시 restart 해줌으로써 안정적으로 실행될 수 있다.

kubeadm 툴은 이와 같은 방법으로 쿠버네티스 클러스터를 구성한다. `kubectl get pods -n kube-system` 명령을 통해서 kube-system의 namespace에서 작동하고 있는 파드를 확인하면, 컨트롤 플레인의 컴포넌트들을 확인할 수 있듯이 말이다.

---
# 스태틱 파드와 데몬셋의 공통점 및 차이점
- 공통점
    - 스태틱 파드와 데몬셋 모두 kube-scheduler에 의해서 만들어지지 않는다는 공통점이 있다.
- 차이점
    - 스태틱 파드는 kube-apiserver 없이, kubelet에 의해서만 생성되는 반면, 데몬셋은 kube-apiserver에 의해 생성된다.
    - 스태틱 파드는 컨트롤 플레인 컴포넌트를 노드에 직접 배포하기 위해서 사용되는 반면, 데몬셋은 모니터링이나 로깅을 위한 에이전트를 배포하기 위해 사용된다.


---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `쿠버네티스 공식 documentation` : [Create static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
