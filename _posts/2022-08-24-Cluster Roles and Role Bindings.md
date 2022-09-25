---
title: (K8S) Cluster Roles and Role Bindings
author: simon sanghyeon
date: 2022-08-24
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Authorization, Cluster Roles]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 네임스페이스 범주와 클러스터 범주
## 네임스페이스 범주에서의 권한 부여
앞서, [Role Based Access Controls(RBAC)](https://zerojsh00.github.io/posts/Role-Based-Access-Controls/)에서 role object와 role binding object를 통해서 역할에 따라 사용자나 그룹의 권한을 부여했다. 이처럼 **RBAC에서의 권한 부여는 특정 namespace에 적용되는 것으로, 특정 namespace를 명시하지 않으면 default namespace에 대해서 role object에 정의된 내용에 따라 권한이 부여**된다. 즉, namespace 단위에 권한을 부여하는 것이므로, 다음과 같이 나열된 namespace 범주에서 관리되는 resource에 대한 권한이 적용되는 것이다.

- **namespace 범주에서 관리되는 resource**
    - 파드
    - 레플리카셋
    - 디플로이먼트
    - 서비스
    - roles
    - rolebindings
    - configmaps
    - PVC
    - etc

## 클러스터 범주에서의 권한 부여
한편, 특정 노드나 PV(Persistent Volume)와 같이, 아래 나열된 cluster 범주에서 관리되는 resource에 대해서 권한을 적용하고자 한다면 어떻게 해야할까?

한편, 특정 노드나 PV(Persistent Volume)와 같이, 아래 나열된 `cluster 범주에서 관리되는 resource`에 대해서 권한을 적용하고자 한다면 어떻게 해야할까?

- **cluster 범주에서 관리되는 resource**
    - 노드
    - PV(Persistent Volume)
    - clusterroles
    - clusterrolebindings
    - certificatesigningrequests
    - namespaces
    - etc

cluster의 범주에서 permission을 주기 위해서는, role object와 role binding object를 이용했던 것과 유사하게, `clusterroles object`와  `clusterrolebindings object`를 정의하고 사용자와 연결해야 한다.

---
# ClusterRole
예를 들어, cluster-admin 계정에게 다음의 권한을 부여하고 싶다고 해보자.

- 노드를 볼 수 있음
- 노드를 생성할 수 있음
- 노드를 삭제할 수 있음

cluster 범주에서 관리되는 resource에 대해서 위와 같은 권한을 부여할 수 있도록 다음과 같이 YAML 파일을 정의할 수 있다.

```yaml
# cluster-admin-role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole # cluster 범주의 resource에 대한 역할 정의이므로 ClusterRole를 입력한다.
metadata:
  name: cluster-administrator # cluster role의 이름을 정의한다.
rules:
- apiGroups: [""]
  resources: ["nodes"] # 클러스터 내 권한을 부여할 자원을 입력한다.
  verbs: ["list", "get", "create", "delete"]
```

이후 `kubectl create -f cluster-admin-role.yaml` 명령을 통해 clusterrole object를 생성할 수 있다.

---
# ClusterRoleBinding
`ClusterRole object`에 정의된 역할을 실제 사용자나 그룹에 적용하기 위해서 사용자와 역할을 연결하는 작업이 필요하다. 이는 아래와 같이 YAML 파일을 정의하여 `ClusterRoleBindings object`를 생성함으로써 가능하다.

```yaml
# cluster-admin-role-binding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding # cluster 범주의 resource에 대한 binding 정의이므로 ClusterRoleBinding을 입력한다.
metadata:
  name: cluster-admin-role-binding # binding 객체의 이름을 입력한다.
subjects:
- kind: User # 적용의 대상이 되는 사용자 정보를 입력한다.
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef: # 적용될 역할 정보를 입력한다.
  kind: ClusterRole
  name: cluster-administrator # 앞서 정의한 ClusterRole의 metadata name을 적는다.
  apiGroup: rbac.authorization.k8s.io
```

이후 `kubectl create -f cluster-admin-role-binding.yaml` 명령을 통해서 ClusterRoleBindings object를 생성할 수 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
