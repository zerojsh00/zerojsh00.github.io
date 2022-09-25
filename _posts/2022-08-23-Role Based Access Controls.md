---
title: (K8S) Role Based Access Controls(RBAC)
author: simon sanghyeon
date: 2022-08-23
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Authorization, RBAC]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

[인가](https://zerojsh00.github.io/posts/Authorization-basic/)의 개념을 다루면서, Role Based Access Controls(RBAC)는 사용자나 그룹의 역할에 따라 permission을 부여하는 방식이라고 정의했다.
그렇다면 쿠버네티스에서는 어떻게 역할을 생성할 수 있을까?

# Role Based Access Controls(RBAC)를 통한 인가 정의하기
우선, 역할이 정의되어 있는 `role object`를 YAML 파일을 이용하여 생성한다.
이후, role object에 정의되어 있는 역할이 특정 사용자나 그룹에 적용될 수 있도록 `사용자와 role object를 연결`시켜야 한다. 이러한 연결 과정은 `role binding object`를 통해서 정의되며, 이 또한 role object와 유사하게 YAML 파일을 이용하여 생성한다.

## Role Object의 정의
예를 들어, 개발자들에게 다음과 같은 권한을 부여하고 싶다고 하자.

- 파드를 볼 수 있음
- 파드를 생성할 수 있음
- 파드를 삭제할 수 있음
- ConfigMaps를 생성할 수 있음

위와 같은 권한을 부여할 수 있도록 다음과 같이 YAML 파일을 정의할 수 있다.

```yaml
# developer-role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer # role object의 이름을 적는다.
rules:
- apiGroups: [""]
  resources: ["pods"] # 파드에 대한 설정을 정의한다.
  verbs: ["list", "get", "create", "update", "delete"]
  resourceNames: ["<파드 이름>"] # 특정 파드만 접근을 허용할 경우 이 행을 작성하며, 생략 또한 가능하다.
- apiGroups: [""]
  resources: ["ConfigMap"] # ConfigMap에 대한 설정을 정의한다.
  verbs: ["create"]
```

이후 `kubectl create -f developer-role.yaml` 명령을 통해서 role object를 생성할 수 있다.


## Role Binding Object의 정의 : 사용자와 Role Object를 연결함
role object에 정의된 역할을 실제 사용자나 그룹에 적용하기 위해서 사용자와 역할을 연결하는 작업이 필요하다.
이는 아래와 같이 YAML 파일을 정의하여 `role binding object`를 생성함으로써 가능하다.

```yaml
# devuser-developer-binding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User # 적용의 대상이 되는 사용자 정보를 입력한다.
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef: # 적용될 역할 정보를 입력한다.
  kind: Role
  name: developer # 앞서 정의한 role object의 metadata name을 적는다.
  apiGroup: rbac.authorization.k8s.io
```

이후 `kubectl create -f devuser-developer-binding.yaml` 명령을 통해서 role binding object를 생성할 수 있다.

위 과정을 마치고 나면, dev-user 계정의 개발자는 default namespace의 파드와 컨피그맵에 접근할 수 있게 된다. 특정 namespace에 대한 접근 permission을 주고 싶다면, metadata 내에서 namespace를 지정함으로써 접근 권한을 줄 수 있다.

---
# View RBAC

생성된 role object들을 보고자 한다면, `kubectl get roles` 명령을 통해서 확인할 수 있다. `kubectl describe role <role object 이름>` 명령으로 role object를 자세히 볼 수도 있다.

생성된 role binding object들을 보고자 한다면, `kubectl get rolebindings` 명령을 통해서 확인할 수 있다. 마찬가지로, `kubectl describe rolebinding <role binding object 이름>` 명령으로 role binding object를 자세히 볼 수도 있다.

---
# Check Access

사용자가 본인이 특정 리소스에 권한이 있는지 확인하고자 한다면, `kubectl auth can-i` 명령어로 확인할 수 있다. 예를 들어, `kubectl auth can-i create deployments` 또는 `kubectl auth can-i delete nodes`와 같이 사용할 수 있으며, 명령어에 대한 결과로 부여된 권한에 따라 yes/no가 출력된다.

관리자 계정으로는 각 사용자들이 특정 리소스에 권한이 있는지도 확인할 수 있다. 위 명령어에 `--as <사용자 계정>`을 덧붙여서 확인할 수 있다. 예를 들어, `kubectl auth can-i create deployments --as dev-user` 또는 `kubectl auth can-i delete nodes --as dev-user`와 같이 사용할 수 있으며, 이에 대한 결과도 부여된 권한에 따라 yes/no가 출력된다. 더 나아가 특정 namespace 하에서의 권한을 확인하고자 한다면 `--namespace <네임스페이스의 이름>`의 옵션을 추가하여 확인할 수도 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
