---
title: (K8S) TLS 보안 기초 개념 정리
author: simon sanghyeon
date: 2022-08-21
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Authentication]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
본 글은 쿠버네티스의 `보안(Security)`을 이해하기에 앞서 필요한, 기초적인 `Transport Layer Security(TLS)` 보안 내용을 정리하는 글이다.

# 대칭 키 암호(Symmetric Encryption)

![fig01](/assets/img/2022-08-21-Security Basics/fig01.png)

사용자가 서버에 접근할 때는 위와 같이 아이디와 비밀번호와 같은 비밀 정보를 전송하여 사용자 인증을 받는다. 이때, 사용자의 비밀 정보는 `암호화`(파란색 박스 부분)되어 서버에 전송된다. 서버에서 암호화된 정보를 `복호화`하기 위해서는 암호화에 사용되었던 `key`를 이용해야 하므로, key 또한 서버에 함께 전송된다. 서버는 이 key를 이용하여 암호화된 사용자 정보를 복호화하여 사용자를 인증한다.

이러한 방식은 암호화와 복호화 과정에서 동일한 key를 사용하는 암호화 방식을 `대칭 키 암호(symmetric encryption)`라고 부른다. symmetric encryption 방식은 암호화와 복호와 과정이 빠르다는 장점이 있다. 그러나 비밀 정보와 key를 네트워크를 통해 전송하는 과정에서 악의적인 목적을 가진 해커가 네트워크 상에서 전송되고 있는 암호화된 정보와 key를 해킹할 수 있다는 문제가 있다. 해커는 비밀 정보를 key를 통해서 해석할 수 있다.

---
# 비대칭 키 암호(Asymmetric Encryption)
symmetric encryption과 같이 암호화와 복호화에 동일한 key를 사용하는 경우 정보 해킹의 문제가 있기 때문에, `비대칭 키 암호(asymmetric encryption)` 방식을 사용할 수 있다. asymmetric encryption에서는 하나의 key만을 사용하는 symmetric encryption과는 달리, `private key`와 `public key`를 구분해서 사용한다. 이 예시에서는 public key를 이용하여 정보를 암호화 할 것이기 때문에, 개념적으로 public lock이라는 표현의 더 어울리기에, `public lock`이라고 부르겠다. 즉, 어떠한 정보를 누구나 접근할 수 있는 public lock을 통해서 암호화 할 수 있지만, 이를 복호화 하기 위해서는 사용자만이 보유할 수 있는 private key가 필요한 것이다.

![fig02](/assets/img/2022-08-21-Security Basics/fig02.png)

예를 들어보자. symmetric encryption의 예제처럼 아이디와 비밀번호를 전송하면 정보 해킹 위험의 소지가 있었다. 그대신 `ssh-keygen` 명령어를 통해서 private key에 해당하는 `id_rsa`와 public key(lock)에 해당하는 `id_rsa.pub` 파일을 생성해서 사용할 수 있다. public lock인 id_rsa.pub 파일을 서버에 설정하고, private key인 id_rsa 정보를 ssh 접속 시 함께 전달해주면, 안전하게 서버에 접근할 수 있는 것이다. 위 그림과 같이 여러 user가 동일한 서버에 접근하는 것도 가능하다. 하나의 서버에 여러 entry를 허용하되, 각 entry에 각각의 public lock을 설정하고, 접근할 때 자신의 private key를 ssh 사용 시 입력해주면 된다.

위와 같이 SSH를 이용해도 좋지만, `OpenSSL`을 이용할 수도 있다. OpenSSL은 네트워크를 통한 데이터 통신에 쓰이는 프로토콜인 TLS와 SSL의 오픈 소스 구현판이다. 예를 들어, `openssl genrsa -out example.key 1024` 명령과 `openssl rsa -in example.key -pubout > example.pem` 명령으로 private key인 `example.key` 파일 및 public key(lock)인 `example.pem`을 생성할 수 있다.

---
# 대칭 키 방식과 비대칭 키 방식을 함께 활용한 암호화와 복호화
![fig03](/assets/img/2022-08-21-Security Basics/fig03.png)

먼저, 사용자가 https를 통해 웹 서버에 접근하면, 사용자는 서버로부터 서버의 public key(lock)를 전송받는다. 그러고 나면, 사용자의 브라우저는 서버의 public key(lock)를 이용하여 사용자의 symmetric key를 암호화한다. 이 과정은 위 그림에서 파란색 박스에 해당한다.

![fig04](/assets/img/2022-08-21-Security Basics/fig04.png)

이후, 서버로 암호화된 symmetric key를 전송할 수 있다. 항상 그렇듯, 해커는 매 순간 트래픽을 보며 정보를 해킹하는데, 이번에는 암호화된 symmetric key가 해킹된다. 그러나 이 경우, 해커는 서버의 private key를 보유하지 못하므로 암호화된 symmetric key를 복호화 할 수 없다. 즉, 사용자의 비밀 정보를 열어볼 방법이 없는 것이다. 반면, 암호화되어 안전하게 서버에 전송된 symmetric key는 서버의 private key에 의해 복호화되고, 사용자의 암호화된 비밀 정보들을 무사히 복호화 할 수 있게 된다.

관행적으로 public key는 `*.crt` 또는 `*.pem` 확장자를 사용한다. 또한 private key는 `*.key` 또는 `*-key.pem`의 확장자를 쓴다.

![fig05](/assets/img/2022-08-21-Security Basics/fig05.png)

그러나 위와 같이 해커가 자신의 서버로 가짜 웹을 호스팅하여 사용자에게 진짜 웹인 것처럼 속인다면, 사용자는 자신의 symmetric key를 해커 서버의 public key(lock)으로 암호화하여 해커의 서버로 전송할 위험이 있다. 이러한 문제를 막기 위해서 `인증서(certificate)`가 필요하다.

---
# 인증서의 개념
사실 사용자로부터 https 접근이 있을 때, 서버는 public key(lock)만을 전송하지 않고, certificate 또한 함께 보낸다. 사용자의 웹 브라우저에는 기본적으로 서버로부터 전송받은 certificate가 유효한지를 검증하는 메커니즘이 존재한다. 유효하지 않은 것으로 보이는 certificate가 발견되면 브라우저에서 `Not secure 경고`를 보내게 된다. 그렇다면, 어떻게 certificate가 유효한지 인증 받을 수 있을까?

암호학에서는 디지털 인증서를 발급하는 곳으로서 `Certificate Authority(CA, 인증 기관)`라고 불리우는 기관이 있는데, 이들에 의해서 유효함을 인증 받을 수 있다. 대표적인 CA로는 `Comodo`, `Semantec`, `DigiCert` 등 여러 곳이 있다.

certificate의 유효함을 인증하기 위해서 `Certificate Signing Request(CSR)`를 요청하면, CA가 CSR을 요청한 곳의 세부 사항을 검증한 후, certificate에 sign을 한다. 이때 세부 사항 검증 과정에서 해커가 발행한 certificate인지를 확인하게 되고, 해커의 경우 CA에 의해 sign이 거절될 수 있다.

한편, 사용자의 브라우저에는 기본적으로 CA의 public key들이 존재한다. 따라서 CA 자체가 가짜인지는 브라우저에서 구별될 수 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
