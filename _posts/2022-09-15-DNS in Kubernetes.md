---
title: (K8S) 쿠버네티스에서의 DNS
author: simon sanghyeon
date: 2022-09-15
categories: [Kubernetes]
tags: [Kubernetes, K8S, Network, DNS]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

![fig01](/assets/img/2022-09-15-DNS in Kubernetes/fig01.png){: width="500" height="500"}

각각의 노드는 `node name`과 `IP 주소`가 부여되어 있으며, 이들은 DNS 서버에 등록되어 있다.

그러나 이번에는 노드에 대한 DNS가 아닌, 쿠버네티스 `클러스터 내부에서 사용 가능한 DNS`를 다루고자 한다. 즉, 클러스터 내에서 파드 간 통신을 할 때 IP가 아닌 도메인을 설정해두고 사용하는 개념을 다룬다.

---
# built-in DNS 서버
![fig02](/assets/img/2022-09-15-DNS in Kubernetes/fig02.png){: width="500" height="500"}

쿠버네티스는 클러스터를 생성할 때 default로 `built-in DNS` 서버를 배포한다. 참고로, 쿠버네티스를 직접 수동으로 설치한다면, 쿠버네티스 DNS 서버 또한 직접 설치해야 한다.

---
# default namespace 하에서

**쿠버네티스 클러스터 내부에서 사용하는 도메인은 서비스와 파드에 대해서 사용할 수 있고, `일정한 패턴`을 가지고 있다.**

![fig03](/assets/img/2022-09-15-DNS in Kubernetes/fig03.png){: width="500" height="500"}

예를 들기 위해서, 노드 단위로는 신경쓰지 않고, 노드 내에 있는 파드와 서비스만을 예시로 다룬다. 위와 같이, default namespace에서 test 파드와 web 파드가 존재하고, test 파드에서 web 파드로 접근하기 위해 web-service 서비스를 만들었다고 해보자. 각각은 모두 IP 주소가 할당되어 있다.

쿠버네티스는 서비스와 파드에 대해서 `DNS record`를 생성한다. 그 덕에 서비스에 접근할 때 IP 주소가 아니라 DNS name으로 접근할 수 있다. 예를 들어, 서비스가 생성될 때마다 쿠버네티스의 DNS 서비스는 위 그림의 표와 같이 `서비스의 이름과 IP 주소를 매핑하는 record`를 생성한다. 따라서 클러스터 내 어떤 파드든지 web-service라는 서비스 이름으로 서비스에 접근할 수 있게 된다.

---
# 또 다른 namespace 하에서

![fig04](/assets/img/2022-09-15-DNS in Kubernetes/fig04.png){: width="500" height="500"}

web 파드와 web-service 서비스가 apps라는 또 다른 namespace에 있다고 가정하자. 이 경우, test 파드에서 web 파드에 접근하기 위해서 web-service를 거친다면, namespace 이름인 `apps`를 마치 이름의 `성(last name)`으로 여기고, web-service`.apps`와 같이 namespace 이름을 명시해주어야 접근할 수 있다.

## 서비스의 DNS record

![fig05](/assets/img/2022-09-15-DNS in Kubernetes/fig05.png){: width="500" height="500"}

서비스의 경우, **`{서비스의 이름}.{namespace의 이름}-svc-cluster.local`**이 도메인 이름이 된다.

풀어 설명하자면, 어떤 namespace 하에 있는 모든 서비스와 파드는 `namespace의 이름(e.g., apps)`이라는 subdomain에 그룹화 된다. 또한 모든 서비스들은 `svc`라고 불리우는 또 다른 subdomain에 그룹화 된다. 마지막으로 모든 파드들과 서비스들은 default로 `cluster.local`로 불리우는 `root` domain에 그룹화 된다. 따라서 위 예시에서 IP 주소가 아닌, 도메인 이름으로 서비스에 접근하고자 하면, `web-service.apps.svc.cluster.local`와 같이 접근할 수 있다.

## 파드의 DNS record

![fig06](/assets/img/2022-09-15-DNS in Kubernetes/fig06.png){: width="500" height="500"}

파드의 경우, **`{파드 IP 주소('.' 대신 '-' 사용)}.{namespace의 이름}-pod-cluster.local`**이 도메인 이름이 된다.

풀어 설명하자면, 파드의 IP 주소를 입력할 때, `.`을 `-`로 대체하여 `10.244.2.5`와 같은 표기 대신 `10-244-2-5`와 같이 사용한다. namespace는 서비스에서처럼 동일하게 namespace 이름을 사용하지만, 타입은 pod가 된다. 이후, 서비스와 동일하게 cluster.local을 붙임으로써 도메인 이름으로 접근할 수 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `쿠버네티스 공식 documentation` : [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)<br>
[4] `아리수 님의 블로그` : [쿠버네티스 DNS(kubernetes dns)](https://arisu1000.tistory.com/27859)
