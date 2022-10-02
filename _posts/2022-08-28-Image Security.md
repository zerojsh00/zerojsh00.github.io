---
title: (K8S) 프라이빗 레지스트리로부터 이미지 받아오기
author: simon sanghyeon
date: 2022-08-28
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Image]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

이번 글은 프라이빗 컨테이너 레지스트리나 레포지터리로부터 이미지를 받아오기 위해 시크릿 객체를 사용하는 방법을 정리하는 글이다.

# 이미지 네이밍의 기본
![fig01](/assets/img/2022-08-28-Image Security/fig01.png){: width="500" height="500"}

파드 정의 YAML 파일에 이미지를 받아오는 부분은 `image: {이미지/레포지터리 이름}`의 형태로 입력한다.
이러한 형태는 도커의 이미지 네이밍 컨벤션을 따르는 것이다. 예를 들어, nginx 이미지를 받아오고자 한다면, image: nginx가 되겠다.

별다른 설정 없이 `image: {이미지/레포지터리 이름}`이라고 입력하면, `image: library/{이미지/레포지터리 이름}`이 된다.
nginx를 예로 들면, `image: library/nginx`가 되겠다. 이때, `library`란 도커의 공식 이미지들이 저장되어 있는 default account의 계정이다.
별도로 사용자/계정을 입력하고자 한다면, library 대신 `image: {사용자/계정 이름}/{이미지/레포지터리 이름}`와 같이 입력하면 된다.

그렇다면, `image: {사용자/계정 이름}/{이미지/레포지터리 이름}`이라는 이미지는 어디에 저장되어 있는 것일까?
별다른 설정이 없으면, 도커의 default registry인 docker hub를 의미하며, `image: docker.io/{사용자/계정 이름}/{이미지/레포지터리 이름}`을 의미하게 된다.
nginx를 예로 들면, `image: docker.io/library/nginx`가 되는 셈이다. docker hub 이외에도 구글의 registry인 gcr.io 등 다양한 registry가 존재한다.

---
# private registry의 이미지 사용하기
## 도커의 경우
AWS, GCP, Azure 등 다양한 클라우드 서비스 제공 업체들이 private registry를 제공한다. 도커의 경우, private 이미지를 실행하기 위해서는 우선적으로 `docker login {private 레지스트리 주소}`를 입력 후 username과 password를 입력하여 로그인을 해야 한다.
그 후, `docker run {private 레지스트리 주소}/{사용자/계정 이름}/{이미지/레포지터리 이름}` 명령어를 통해 실행한다.
예를 들면, `docker run private-registry.io/apps/internal-app`과 같이 실행할 수 있겠다.

## 쿠버네티스의 경우 : docker-registry 시크릿 객체
쿠버네티스로 돌아와서, 이를 파드 정의 YAML 파일에 입력하기 위해서는 어떻게 해야 할까?
`image : {private 레지스트리 주소}/{사용자/계정 이름}/{이미지/레포지터리 이름}`와 같은 방식으로 이미지 경로는 입력할 수 있겠으나, 도커에서 로그인 정보를 입력했던 것처럼 비밀 정보(즉, authentication 문제)들은 어떻게 처리해야 할까?

이를 위해서는 `시크릿 객체`를 생성해서 username이나 password 같은 정보들을 입력해주어야 한다.
이 예에서는 시크릿 객체의 이름을 ‘regcred’라 하겠다. **아래와 같이 `docker-registry`라는 시크릿을 이용해서 도커의 credential을 입력한다.
docker-registry는 built-in 시크릿 타입으로, 도커의 credential들을 저장하기 위해서 만들어졌다.**

![fig02](/assets/img/2022-08-28-Image Security/fig02.png){: width="500" height="500"}

이로써, regcred라는 이름의 시크릿으로 도커 credential을 입력하였다. 이후 아래와 같이 imagePullSecrets에 시크릿의 이름을 입력해주면 된다.

![fig03](/assets/img/2022-08-28-Image Security/fig03.png){: width="500" height="500"}

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
