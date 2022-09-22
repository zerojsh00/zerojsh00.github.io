---
title: (K8S) 테인트와 톨러레이션 vs 노드 어피니티
author: simon sanghyeon
date: 2022-08-16
categories: [Kubernetes]
tags: [Kubernetes, K8S, Scheduling, Taint, Toleration, NodeAffinity]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

이번 글에서는 파드를 적합한 노드에 배포하기 위해 [테인트와 톨러레이션](https://zerojsh00.github.io/posts/K8S_Taints-and-Tolerations/)과 [노드 어피니티](https://zerojsh00.github.io/posts/Node-Selector-and-Node-Affinity/)를 조합하여 사용하는 방법을 정리한다.

# 문제의 정의

다음과 같이 쿠버네티스 클러스터를 다른 팀과 함께 공유해서 사용한다고 가정해보자.

![fig01](/assets/img/2022-08-16-Taints and Tolerations vs Node Affinity/fig01.png)

최종 목표로, 우리 팀 파드인 파드 A, 파드 B, 파드 C는 색깔에 맞게 적합한 노드에 배치하고, 타 팀의 파드는 우리 팀의 노드에 배치되지 않도록 설정하고자 한다.

---

# 테인트와 톨러레이션만 사용하는 경우

다음과 같이 테인트와 톨러레이션을 사용하면 우리 팀의 파드를 적합한 노드에 배치하는 것이 가능하다. 이는 이상적인 경우다.

![fig02](/assets/img/2022-08-16-Taints and Tolerations vs Node Affinity/fig02.png)
*테인트와 톨러레이션만을 사용하는 예 - 이상적인 경우*

그런데 문제는 테인트와 톨러레이션만을 이용하면, 다음과 같이 테인트되지 않은 다른 팀의 노드에 우리 팀의 파드가 스케줄링 될 수 있다.

![fig03](/assets/img/2022-08-16-Taints and Tolerations vs Node Affinity/fig03.png)
*테인트와 톨러레이션만을 사용하는 예 - 문제가 발생한 경우*

---

# 노드 어피니티만 사용하는 경우

노드 어피니티를 사용하면, 우리 팀의 파드를 우리 팀 소유의 특정 노드에만 배치될 수 있도록 제한할 수 있다. 이 또한 이상적인 경우다.

![fig04](/assets/img/2022-08-16-Taints and Tolerations vs Node Affinity/fig04.png)
*노드 어피니티만을 사용하는 예 - 이상적인 경우*

그런데 문제는, 노드 어피니티만을 사용한다면, 다음과 같이 타 팀의 파드가 우리 팀 소유의 노드에 배치되는 것을 막을 수 없다.

![fig05](/assets/img/2022-08-16-Taints and Tolerations vs Node Affinity/fig05.png)
*노드 어피니티만을 사용하는 예 - 문제가 발생한 경우*

---

# 테인트와 톨러레이션, 그리고 노드 어피니티를 함께 사용하는 경우

따라서 테인트와 톨러레이션, 그리고 node affinity를 동시에 사용해야 목표를 달성할 수 있다.

![fig06](/assets/img/2022-08-16-Taints and Tolerations vs Node Affinity/fig06.png)
*테인트와 톨러레이션, 그리고 노드 어피니티를 함께 사용하는 경우*

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
