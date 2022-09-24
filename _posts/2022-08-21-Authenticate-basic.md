---
title: (K8S) 쿠버네티스 인증(Authentication) 기초 개념
author: simon sanghyeon
date: 2022-08-21
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Authentication]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
본 글에서는 쿠버네티스의 `인증(Authentication)` 매커니즘의 가장 기초적인 방법을 단순히 공부 목적으로 정리한다. 따라서 실전에서 추천되는 방법은 아니며, 기본적인 개념 정도만 잡고 가면 좋을 것 같다.

# 인증(Authenticate)
`인증(Authentication)`이란, 사용자가 누구인지를 식별하는 행위로, 우리가 흔히 사용하는 로그인 기능도 일종의 인증 과정이라고 생각할 수 있다.

쿠버네티스에서는 `사용자 계정(User Account)`과 `서비스 계정(Service Account)`에 대해 인증한다.
사용자 계정에 대한 인증은 우리가 일반적으로 생각하는 사용자를 인증하는 개념이며, 서비스 계정에 대한 인증은 클라이언트나 기타 컴포넌트가 kube-apiserver를 호출하는 등 쿠버네티스에서의 통신에 있어 시스템(e.g., Bot)을 인증하는 개념이다.

쿠버네티스는 근본적으로 사용자 계정을 직접적으로 관리하는 시스템이 존재하지 않기 때문에, `kubectl create user {사용자 이름}`과 같은 명령어 또한 존재하지 않는다.
따라서 근본적으로 사용자를 직접 관리할 수 없는 쿠버네티스에서는 `사용자 정보 상세가 적혀있는 파일`이나 `certificates`, 또는 `OAuth`나 `Webhook`과 같은 별도의 외부 계정 연동 시스템을 사용해야 한다.
한편, 서비스 계정은 이처럼 관리할 수는 있다.

## 기초적인 방식 : 사용자 정보 상세 파일을 이용
사용자를 인증하기 위한 예시로, 가장 기초적인 방식인 `사용자 정보 상세가 적혀있는 파일`의 경우를 살펴보겠다. 실제 운영 시에는 적합하지 않으므로 개념만 잡고 가도 좋을 것이다.

아래의 예시는 그중에서도 패스워드를 이용하는 `static password file`을 적용하는 사례인데, 이외에도 토큰을 활용하는 static token file을 적용할 수도 있다.

![fig01](/assets/img/2022-08-21-Authenticate-basic/fig01.png)

위와 같이 사용자의 패스워드, 사용자 이름, 사용자 아이디(네번째 열에 `그룹` 추가 가능)가 적혀있는 user-details.csv 파일을 생성한 이후, kube-apiserver.service에 `--basic-auth-file=user-details.csv`를 입력하여 설정할 수 있다. 이후, kube-apiserver를 재실행하면 사용자 인증이 적용된다.

![fig02](/assets/img/2022-08-21-Authenticate-basic/fig02.png){: width="500" height="500"}

kubeadm 툴을 이용한다면, 위와 같이 설정해주면 되는데, 이 파일이 업데이트되면 자동으로 kube-apiserver가 재실행되어 반영된다.

사용자 인증 설정이 완료되면, 다음과 같이 사용자 정보와 패스워드를 입력하여 kube-apiserver에 접근할 수 있게 된다.

![fig03](/assets/img/2022-08-21-Authenticate-basic/fig03.png){: width="500" height="500"}

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `조대협 님의 블로그` : [쿠버네티스 #16 - 보안 (1/4) 계정 인증과 권한 인가](https://bcho.tistory.com/1272)
