---
title: (K8S) 서비스(Service) 타입 - ClusterIP
author: simon sanghyeon
date: 2022-08-08
categories: [Kubernetes]
tags: [Kubernetes, K8S, Service]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

앞서 쿠버네티스 환경에서 [서비스(Service)의 개요와 종류](https://zerojsh00.github.io/posts/K8S_Service/)를 간단히 살펴보았다.
이번에는 서비스의 종류 중 `ClusterIP` 서비스를 살펴보고자 한다.

# ClusterIP 서비스란?

`ClusterIP` 타입은 쿠버네티스가 지원하는 기본 형태의 서비스로, 파드들이 클러스터 내부의 다른 리소스들과 통신할 수 있게 하는 `가상의 클러스터 전용 IP`다.
ClusterIP는 클러스터 내에서만 사용되는 IP이므로 외부에서는 접근할 수 없다는 점을 유의해야 한다. 클러스터 내부에서만 사용하는 파드라면 상관 없겠으나, 외부에 노출되어야 한다면 NodePort나 LoadBalancer 타입의 서비스를 사용해야 한다.

---

# ClusterIP 서비스 사용의 예

풀스택 웹 애플리케이션을 예로 들어보면, 애플리케이션의 각 부분, 즉 프론트엔드 서버, 백엔드 서버, 그리고 인메모리 key-value 스토어인 Redis나 RDB MySQL 같은 데이터베이스 등이 서로 다른 파드들로 구성되어 있다.
이러한 구조에서 애플리케이션들은 ClusterIP를 통해 통신할 수 있다.

![fig01](/assets/img/2022-08-08-K8S_Service-ClusterIP/fig01.png)

위와 같은 구조를 예로 들면, ClusterIP는 각 tier에 있는 파드들을 하나로 그룹핑하고, 다른 tier의 파드 그룹과 소통할 수 있도록 `하나의 통합된 interface`(예를 들어, 그림에서 back-end 파드들에 접근하기 위한 서비스)를 제공한다.
이때 각 서비스는 고유한 IP 주소인 ClusterIP를 가진다.

이러한 방식은 쿠버네티스 클러스터 상에서 마이크로서비스 기반 애플리케이션을 효과적으로 배포할 수 있게 한다.
그덕에 각 레이어에 있는 파드들은 다양한 서비스를 통해 통신되므로, 커뮤니케이션에 지장을 주지 않으면서 스케일링 될 수 있는 것이다.

---

# ClusterIP 정의하기

```yaml
# service-definition.yml

apiVersion: v1
kind: Service
metadata:
  name: back-end

spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    port: 80
    # nodePort 항목이 없음을 주의! 클러스터 내부에서만 사용되며 외부로 노출되기 않기 때문임
  selector: # pod 생성 시 labels에 정의되어 있던 내용
    app: myapp
    type: back-end
```

`kubectl create -f service-definition.yaml` 명령으로 생성하며, `kubectl get services` 명령으로 확인한다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
