---
title: (K8S) 디플로이먼트
author: simon sanghyeon
date: 2022-08-07
categories: [Kubernetes]
tags: [Kubernetes, KL8S, Deployment]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

앞서 [레플리케이션 컨트롤러와 레플리카셋](https://zerojsh00.github.io/posts/K8S_ReplicaSet/)에 대해서 살펴보았다.
그리고 최근에는 이들 중 레플리카셋을 사용한다는 것도 살펴보았다. 그러나 실제 쿠버네티스 운영 환경에서는 YAML 파일을 이용해서 레플리카셋을 사용하는 경우는 매우 드물다.
대부분 레플리카셋과 파드의 정보를 정의하는 `디플로이먼트(deployment)`를 YAML 파일로 정의하여 사용한다.

# Deployment
`디플로이먼트(deployment)`는 레플리카셋의 상위 개념인 오브젝트이다. 따라서 디플로이먼트를 생성하면 해당 디플로이먼트에 정의된 레플리카셋도 함께 생성된다. 즉, 별도로 레플리카셋을 생성할 필요가 없어진다.

---

# Deployment의 YAML
디플로이먼트의 YAML 파일은 레플리카셋의 YAML 파일과 매우 비슷하다. 단지 kind 및 metadata 섹션만 Deployment로 바꾸면 된다.

```yaml
# deployment-definition.yaml

apiVersion: apps/v1 # Replication Controller와 다른 점을 주의하자! apps/를 붙여야한다!
kind: Deployment # Replication Set과 다른 부분! 이것만 바꾸면 된다.
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end

spec: # replica set를 위해서 필요한 spec
  template:
    # 이곳에는 replica set에 의해서 사용될 파드의 템플릿을 기입한다.
    # 파드 생성에 필요했던 YAML파일에서 apiVersion과 kind 빼고 복사 붙여넣기 해온다.
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec: # pod에 대한 spec
        containers:
        - name: nginx-container
          image: nginx

  replicas: 3 # how many replicas

  # Replication Controller와 다른 점
  selector: # The selector section helps the replica set identify what pods fall under it.
    matchLabels:
      type: front-end
```

디플로이먼트 정의 YAML 파일 작성이 완료되면, `kubectl create -f deployment-definition.yaml` 명령을 통해서 생성한다.

`kubectl get deployments` 및 `kubectl get replicaset`으로 디플로이먼트 및 레플리카셋 정보를 확인할 수 있다.
특히 레플리카셋 정보를 확인해보면, myapp-deployment-6795844b58과 같이 deployment이름으로 replicaset이 구동됨을 알 수 있다.
`kubectl get all` 명령을 통해 디플로이먼트, 레플리카셋, 파드 정보를 한번에 확인할 수 있다.

---

# Deployment를 사용하는 이유
쿠버네티스는 왜 레플리카셋을 굳이 상위 개념인 디플로이먼트를 통해서 사용하는 것일까?
디플로이먼트를 사용하는 주된 이유는 애플리케이션의 업데이트와 배포를 효과적으로 진행하기 위해서다.

디플로이먼트는 무중단 서비스를 위해 파드의 `롤링 업데이트(rolling update)`를 지원한다.
또한 업데이트될 때 레플리카셋의 변경 사항을 저장하는 `리비전(revision)`을 남기며, 업데이트 중 예상치 못한 장애가 발생했을 때는 리비전을 통해 `롤백`이 가능하도록 지원한다.
이러한 장점으로 인해 쿠버네티스에서도 공식적으로 디플로이먼트를 사용할 것을 권장한다.

---

# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
