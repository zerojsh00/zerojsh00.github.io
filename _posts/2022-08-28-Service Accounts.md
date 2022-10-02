---
title: (K8S) 쿠버네티스 서비스 계정(Service Accounts)
author: simon sanghyeon
date: 2022-08-28
categories: [Kubernetes]
tags: [Kubernetes, K8S, Security, Authentication]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---

쿠버네티스에는 두 종류의 계정(account)이 존재하는데, 하나는 사람이 사용하는 `user account`이며, 다른 하나는 기계나 bot 등이 사용하는 `service account`이다. user account의 예로는 클러스터를 관리하는 관리자 계정이나 애플리케이션을 배포하는 개발자 계정 등이 있다. service account의 예로는 프로메테우스(prometheus)와 같은 모니터링 애플리케이션, 애플리케이션 배포를 위한 빌드 자동화 툴인 젠킨스(Jenkins) 등이 있다.

# Service Account와 인증을 위한 토큰

![fig01](/assets/img/2022-08-28-Service Accounts/fig01.png){: width="500" height="500"}

service account에 대해 더욱 살펴보자. 예를 들어, K8S 대시보드라는 애플리케이션을 만들었다고 가정해보자. 이 애플리케이션은 kube-apiserver에 request를 보내서 파드 정보를 받아온 후, 가져온 정보들을 웹을 통해 나열해주는 간단한 기능을 제공한다고 해보자. **이 경우, K8S 대시보드 애플리케이션은 쿠버네티스 외부의 third-party application에 해당하므로, kube-apiserver에 query를 보내기 위해서는 클러스터에 접근할 수 있도록 인증(authentication)이 필요하다. 이때 service account가 사용된다.**

service account를 생성하는 방법은 `kubectl create serviceaccount {service account의 이름}` 명령어를 입력하여 생성할 수 있다.
또한, `kubectl get serviceaccount` 명령을 통해 현재 생성되어 있는 service account를 확인할 수 있다.

```bash
[node1 ~]$ kubectl create serviceaccount dashboard-sa
serviceaccount/dashboard-sa created

[node1 ~]$ kubectl get serviceaccount
NAME           SECRETS   AGE
dashboard-sa   1         7s
default        1         2m45s
```

`kubectl describe serviceaccount {service account의 이름}` 명령을 통해서 다음과 같이 service account를 자세히 확인할수도 있다.
```bash
[node1 ~]$ kubectl describe serviceaccount dashboard-sa
Name:                dashboard-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   dashboard-sa-token-gtb2q
Tokens:              dashboard-sa-token-gtb2q
Events:              <none>
```

특이한 점으로, `Tokens: dashboard-sa-token-gtb2q`와 같이 토큰이라는 정보가 생성되어 있는 것을 확인할 수 있다.
**service account의 객체가 생성되면, 해당 객체의 `service account 토큰`이 자동으로 만들어진다. 이 토큰은 쿠버네티스 외부의 애플리케이션이 kube-apiserver에 접근할 때 인증하기 위해 사용된다. 이후 이 토큰을 저장하기 위한 `시크릿 객체`가 만들어지며, 토큰은 시크릿 객체 내부에 저장된다. 토큰이 저장된 시크릿 객체는 service account와 연동된다.**

토큰의 정보는 `kubectl describe secret {secret 객체에 저장되어 있는 토큰의 이름}` 명령을 통해서 확인할 수 있다.

```bash
[node1 ~]$ kubectl describe secret dashboard-sa-token-gtb2q
Name:         dashboard-sa-token-gtb2q
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: 56802a9f-c87f-43bb-ac5f-6c7eb104e960

Type:  kubernetes.io/service-account-token

Data
====
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImt ... 생략 ...
```

앞서, 이 토큰 정보는 쿠버네티스 API에 접근할 때 인증하기 위해서 사용된다고 했다.
예를 들면, `curl https://192.168.56.70:6443/api -insecure --header "Authorization: Bearer eyJhbGci0iJSUzI1NiIsImt ... 생략 ..."`과 같이 사용할 수 있다.

이 예에서 예로 들은 K8S 대시보드와 같은 third-party application에서는 service account 토큰을 컨피그로 지정해서 사용할수도 있다.

---
# default service account

한편, `kubectl get serviceaccount` 명령을 입력해보면, 다음과 같이 `default service account`가 생성되어 있는 것을 확인할 수 있다.

```bash
[node1 ~]$ kubectl get serviceaccount
NAME           SECRETS   AGE
dashboard-sa   1         7s
default        1         2m45s
```

default service account와 이에 대한 토큰은 파드가 생성될 때마다 자동으로 해당 파드에 volume mount 형태로 마운트 된다. 예를 들어서 아래와 같이 파드를 만든다고 해보자.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
  - name: my-kubernetes-dashboard
    image: mykubernetes-dashboard
```
위의 파드 정의 YAML 파일에는 volume mount라든지 시크릿에 관한 어떠한 설정도 정의해두지 않았다.
그러나 이 파드를 생성한 후, `kubectl describe pod my-kubernetes-dashboard`명령을 통해서 해당 파드를 살펴보면, volumes 부분에 default 토큰이 시크릿 타입으로 마운트되어 있는 것을 확인할 수 있다.
해당 토큰에 대한 정보는 `/var/run/secrets/kubernetes.io/serviceaccount` 경로에서 확인할 수 있으며, `kubectl exec -it my-kubernetes-dasyboard cat /var/run/secrets/kubernetes.io/serviceaccount/token`와 같이 토큰을 확인해 볼수도 있다.

한편, 이러한 default service account는 인증이 매우 제한적이기 때문에, 간단한 쿠버네티스 API 쿼리에만 permission이 있다.

---
# 직접 생성한 service account 사용하기
default service account가 아닌, 직접 만든 service account를 사용하기 위해서는 아래와 같이 파드 설정 YAML 파일에 `serviceAccountName 설정`을 추가해주면 된다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
  - name: my-kubernetes-dashboard
    image: mykubernetes-dashboard
serviceAccountName: dashboard-sa # 직접 생성한 service account의 이름을 입력한다.
```

serviceAccountName 설정을 변경한 이후, 변경 사항을 파드에 적용하기 위해서는 반드시 해당 파드를 제거한 후 재생성해야 한다.
한편, 디플로이먼트의 경우, 파드 정의 YAML 파일에 serviceAccountName을 변경하면 자동으로 변경이 적용된다.

---
쿠버네티스가 자동으로 default service account를 생성해서 파드에 마운트하는 것을 원치 않는다면, 아래와 같이 파드 정의 YAML 파일에 `automountServiceAccountToken 설정`을 추가해주면 된다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
  - name: my-kubernetes-dashboard
    image: mykubernetes-dashboard
automountServiceAccountToken: False # default service account 토큰이 파드에 자동으로 마운트되지 못하게 한다.
```
---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
