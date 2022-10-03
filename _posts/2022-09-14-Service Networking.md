---
title: (K8S) 서비스 네트워킹
author: simon sanghyeon
date: 2022-09-14
categories: [Kubernetes]
tags: [Kubernetes, K8S, Network, Service]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
이전 글들에서 파드 간에 네트워크 연결이 될 수 있도록 많은 개념들을 정리했지만, 사실 파드들끼리 직접적으로 연결되도록 설정할 일은 많이 없고, 그대신 서비스(서비스)를 이용해야 한다.

# Recap: 서비스

![fig01](/assets/img/2022-09-14-Service Networking/fig01.png){: width="500" height="500"}

다음과 같이 노드 `NODE1`, `NODE2`, `NODE3`가 있고, 각 노드 안에는 `파드(동그라미)`들이 존재한다고 하자. 자신이 아닌 다른 파드에서 호스트되는 서비스를 이용하고 싶다면, 반드시 해당 파드의 `서비스(그림에서 세모)`를 통해 접근해야 한다. 예를 들어, NODE1의 파란색 파드가 주황색 파드에 접근하고자 하면, 반드시 `주황색 서비스`를 생성하고 이를 통해 접근해야 한다. 이 경우, 주황색 서비스는 별도의 IP(10.99.13.178)가 부여되는 `ClusterIP 타입`의 서비스이다. 서비스는 클러스터 내에서 모두가 접근할 수 있으므로, 다른 노드에 있는 파드들 또한 주황색 서비스를 통해서 접근할 수 있게 된다.

한편, 보라색 파드는, 가령 웹 애플리케이션과 같이, 클러스터 내부 뿐만 아니라 외부에서도 접근이 되어야 하는 파드라고 해보자. 이 경우, `보라색 서비스` 또한 ClusterIP 타입과 유사하게 IP(10.99.13.178)를 가지는 서비스이며, 클러스터 내부에서 모두 보라색 서비스를 거쳐 접근할 수 있다. 다른 점은 클러스터 외부에서 접근할 수 있도록 클러스터 내 모든 노드에서 특정 `포트(30080)`를 개방하고 있다. 이 경우, 보라색 서비스는 `NodePort 타입`의 서비스이다.

이번글은 **어떻게 서비스가 위와 같이 IP 주소를 할당 받을 수 있으며, 클러스터 내부에서 접근될 수 있는지, 그리고 NodePort 타입의 경우 각 노드에서 어떻게 포트를 통해 외부 사용자가 접근할 수  있는지** 등을 간단하게 짚고 간다.

---
![fig02](/assets/img/2022-09-14-Service Networking/fig02.png)

# Kubelet과 파드

쿠버네티스의 모든 노드에는 `kubelet`이 하나씩 존재하며, 각 kubelet은 kube-apiserver와 통신하며 파드를 노드에 생성한다. **kubelet이 파드를 생성할 때, 해당 파드의 네트워크를 구성하기 위해 CNI 플러그인을 호출한다.**

# Kube-proxy와 서비스

`kube-proxy`는 쿠버네티스에서 서비스를 만들었을 때 Cluster IP나 NodePort로 접근할 수 있게 하는 실제 조작을 수행하는 컴포넌트다.

쿠버네티스의 각 노드에는 kube-proxy 또한 하나씩 존재하며, 각 kube-proxy는 kube-apiserver와 통신하며 서비스를 노드에 생성한다. 파드와는 다르게, 서비스는 `cluster-wide`한 개념이므로, 특정 노드에 할당되어 생성되지는 않으며, 클러스터 전반에 걸쳐서 존재하는 개념이다.

엄밀하게 말하자면, 서비스는 그 실체가 없다고 볼 수도 있다. 파드와 서비스를 비교했을 때, 파드는 컨테이너 네임스페이스가 별도로 존재하고, IP가 할당된 인터페이스도 있지만, 서비스에는 그런 개념이 없다. 서비스는 단지 가상 `객체(virtual object)`일 뿐이다.

그렇다면 어떻게 서비스에 IP 주소가 부여되고, 서비스를 통해서 파드에 있는 애플리케이션에 접근할 수 있는 것일까?

**우선, 사전에 정의되어 있는 서비스의 `IP:Port`의 범위(predfined range)들이 있다고 하자. 각 노드에서 실행되고 있는 kube-proxy 컴포넌트는 IP:Port(그림에서 10.99.13.178:80) 정보가 들어오면, 사전에 정의되어 있는 서비스의 IP:Port 범위에 따라 해당 서비스의 IP:Port에 대응되는 클러스터 내 특정 노드의 파드(그림에서 10.244.1.2)로 전달하는 `forwarding rule`을 정의하여 사용한다.**

# Forwarding Rule

IP:Port 트래픽이 있을 때, kube-proxy가 특정 노드의 파드로 요청을 전달하기 위한 관리 방법으로는 `iptables`(default proxy mode), `userspace`, `IPVS`가 있다.


---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `김징어 님의 블로그` : [[k8s] kube-porxy가 네트워크를 관리하는 3가지 모드(userspace, iptables, IPVS)](https://kimjingo.tistory.com/152)
