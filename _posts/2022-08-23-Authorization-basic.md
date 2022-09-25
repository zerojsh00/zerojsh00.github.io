---
title: (K8S) 인가(Authorization)의 기초 개념
author: simon sanghyeon
date: 2022-08-23
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Authorization]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 인가(Authorization)의 개념
[인증(authentication)](https://zerojsh00.github.io/posts/Authenticate-basic/) 과정을 통해 누가 클러스터에 접근할 수 있을지를 결정했다면, 어떤 일을 수행할 수 있는지 권한에 대한 정의가 필요한데, 이를 `인가(authorization)`이라고 한다.

![fig01](/assets/img/2022-08-23-Authorization-basic/fig01.png)

위의 상황과 같이, `admin` 계정, `developer` 계정, 그리고 `bot`이 존재한다고 예를 들어 보자. 이때 bot은 모니터링 애플리케이션이나 Jenkins와 같은 지속적 배포 도구 등으로 생각하자. 이 경우, 모든 계정에게 동일한 권한을 주면 위험할 수 있다. 관리자인 admin 계정은 모든 권한을 가질 수 있겠으나, 개발자인 developer 계정이 노드를 삭제한다는 등 자신의 권한을 넘어서는 행동들은 제한되어야 한다. 즉, authorization이 필요하다.

쿠버네티스에서 authorization을 부여하는 메커니즘은 다양하게 존재한다. 대표적으로 `Node Authorization`, `Attribute-based Authorization(ABAC)`, `Role-based Authorization(RBAC)`, 그리고 `Webhook`이 있다. 추가적으로 `AlwaysAllow` 및 `AlwaysDeny` 모드 또한 존재한다.

## Node Authorizer

kubelet은 현재 노드의 상태와 같은 정보들을 kube-apiserver에 전송하는데, 이때의 요청을 처리하는 authorizer가 바로 `Node Authorizer`이다. 간단하게, Node Authorization은 kubelet의 API 요청을 인증하는 목적으로 사용되는 특수한 authorization이라고 보고 넘어가자.

## ABAC(Attribute-based Access Control)

**ABAC는 사용자의 속성(user attribute) 기반의 authorization으로, 사용자 개인이나 사용자 그룹에 permission을 부여하는 데 사용된다.** 예를 들어, 어떤 개발자A의 계정에 `view PODs`, `create PODs`, `delete PODs`를 할 수 있는 권한을 부여한다고 하면, `{"kind": "policy", "spec": {"user": "dev-user", "namespace": "*", "resource": "pods", "apiGroup": "*"}}`과 같이 JSON 형식으로 policy file을 정의한 후, 해당 policy file을 API server에 전달함으로써 권한을 부여할 수 있다. 같은 방식으로 개발자B, 개발자C 등에 대해서도 일일이 policy file을 정의할 수 있다.

그러나 이러한 방식은 permission 부여에 변화가 있을 때마다 매번 policy file을 직접 수정해야 하며, API server를 재실행해주어야 한다. 즉, 관리가 번거로운 단점이 있다.

## RBAC(Role-based Access Control)

**ABAC 방식의 단점을 보완할 수 있는 RBAC는 사용자나 그룹에 직접 하나하나 permission을 정의하는 방식이 아닌, 사용자나 그룹의 역할에 따라 permission을 정의하는 방식이다.** 예를 들어, 개발자 계정들에 대해서는 `view PODs`, `create PODs`, `delete PODs`에 대한 permission을 한꺼번에 부여해주고, security 관련 계정에게는 `view CSR`, `approve CSR`에 대한 permission을 한꺼번에 부여해주는 식으로 관리한다.

RBAC는 쿠버네티스에서 access를 관리하는 방식의 표준이며, 이와 관련해서는 추후에 더욱 깊이 다룬다.

## Webhook

쿠버네티스 상에 built-in 되어있는 authorization 메커니즘이 아니라, 서드파티와 같은 외부로부터 authorization 방식을 아웃소싱하고 싶을 때 활용되는 것이 webhook 방식이다.

## AlwaysAllow & AlwaysDeny

이름에서 직관적으로 알 수 있듯이, `AlwaysAllow`는 항상 permission을 허용하는 방식이며, `AlwaysDeny`는 항상 permission을 거부하는 방식이다.

---
# Authorization Mode의 설정
![fig02](/assets/img/2022-08-23-Authorization-basic/fig02.png)

`Node Authorization`, `Attribute-based Authorization(ABAC)`, `Role-based Authorization(RBAC)`, `Webhook`, `AlwaysAllow` 및 `AlwaysDeny` 모드의 설정은 kube-apiserver의 authorization mode에서 설정할 수 있다. 모드가 여러 개로 설정되어 있는 경우, 순차적으로 각 모드를 적용해보았을 때, permission이 허용되는 모드가 작동되며, 그 이외의 모드는 작동되지 않는다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
