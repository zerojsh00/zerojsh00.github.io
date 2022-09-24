---
title: (K8S) 쿠버네티스 환경변수 설정, 컨피그맵(ConfigMap)과 시크릿(Secret)
author: simon sanghyeon
date: 2022-08-19
categories: [Kubernetes]
tags: [Kubernetes, K8S, Docker, Configure, ConfigMap, Secret]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 쿠버네티스 환경변수 설정
우리가 개발한 애플리케이션은 대부분 여러가지 환경변수(설정값)들을 정의해야 한다.
간단한 예를 들어보면, 로깅 레벨을 정의할 때 `LOG_LEVEL=INFO`처럼 단순히 `키-값 형태의 설정값`이 되든지, 아니면 Nginx 웹 서버가 사용하는 nginx.conf처럼 `완전한 하나의 설정 파일 형태`가 되든지 환경변수를 정의해야 한다.

이러한 환경변수를 우리의 애플리케이션에 전달하는 가장 확실하고 직관적인 방법은 도커 이미지 내부에 설정값 또는 설정 파일을 정적으로 저장해두는 것이다.
그러나 이러한 방식은 단점이 있는데, 도커 이미지는 한번 빌드가 완료되면 불변의 상태이기 때문에 설정 옵션을 유연하게 변경할 수 없다.

## 파드 정의 YAML 파일을 활용한 환경변수 설정
쿠버네티스에서는 다음과 같이 파드 정의 YAML 파일에서 `env`를 통해 컨테이너에 환경변수를 직접 하드 코딩 방식으로 전달할 수 있다.
```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080

      # 아래 env 부분에서 환경 변수를 설정한다.
      env:
        - name: APP_COLOR # 컨테이너가 사용할 수 있는 환경 변수의 이름
          value: pink # 환경 변수의 값
```

이와 같은 방식을 `Plain Key Value`라고 부른다. 나쁜 방식은 아니지만, **상황에 따라서 환경변수의 값만 다르고 내용은 동일한 YAML 파일을 여러 개 사용해야 하는 경우도 있다.**
이 경우, 파드 정의 YAML 파일에서 환경변수만을 분리해서 관리하면 한결 편리할 것이다.

쿠버네티스는 YAML 파일과 환경변수를 분리할 수 있는 방식으로 `컨피그맵(ConfigMap)`과 `시크릿(Secret)` 오브젝트를 제공한다.

---
# 컨피그맵(ConfigMap)
앞서 파드 정의 YAML 파일에서 환경 변수를 정의하는 방식을 살펴보았다. 그런데, YAML 파일이 많아지면, 다양한 파일들에 정의되어 있는 환경변수들 또한 관리하기 어려워진다.
따라서 파드 정의 YAML 파일 밖에서 환경변수들을 관리하는 방식인 `컨피그맵(ConfigMap)`이 필요하다.

**컨피그맵은 환경변수 데이터를 키-값(key-value)의 쌍으로 전달하기 위해 사용된다.** 컨피그맵은 파드가 생성될 때 파드에 주입되고, 이렇게 주입된 키-값 쌍의 환경변수들은 파드 내 컨테이너에서 호스트되는 애플리케이션에서 접근할 수 있게 된다.
즉, 다음 두 단계를 거치는 것이다.

- 컨피그맵 생성하기
- 파드에 컨피그맵 주입하기

## 컨피그맵 생성하기
컨피그맵은 다른 쿠버네티스 오브젝트들과 마찬가지로 `명령형(imperative) 방식`과 `선언형(declarative) 방식`으로 생성할 수 있다.

### 컨피그맵 생성하기 : 명령형(imperative) 방식

- `kubectl create configmap {컨피그맵 이름} --from-literal={키}={값}`
  - 예) `kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod`

위와 같이 `--from-literal` 옵션을 사용하여 컨피그맵에 환경변수를 키-값 형태로 전달할 수 있다. 또한, `--from-literal` 옵션을 여러 번 사용하여 여러 개의 요소들을 한번에 입력할 수도 있다.


- `kubectl create configmap {컨피그맵 이름} --from-file={파일의 경로}`
  - 예) `kubectl create configmap app-config --from-file=app_config.properties`

위와 같이 파일의 경로를 지정해주는 방식도 가능하다.

참고로, 파일로 사용한 app_config.properties는 아래와 같이 구성할 수 있다.

```
# app_config.properties

APP_COLOR=blue
APP_MODE=prod
```

### 컨피그맵 생성하기 : 선언형(declarative) 방식

```yaml
# config-map.yaml

apiVersion: v1
kind: ConfigMap # kind는 ConfigMap으로 설정한다.
metadata:
  name: app-config
data: # 아래 부분에 ConfigMap 내용을 적어준다.
  APP_COLOR: blue
  APP_MODE: prod
```

이후 `kubectl create -f config-map.yaml` 명령어를 입력한다.

`kubectl get configmaps` 또는 `kubectl describe configmaps` 명령어를 통해 현재의 ConfigMap을 확인할 수 있다.

## 파드에 컨피그맵 주입하기
컨피그맵을 생성했다면, 생성된 컨피그맵을 파드에 주입해야 우리가 따로 설정한 환경변수를 활용할 수 있다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080

      # 아래 envFrom 부분을 통해 생성된 ConfigMap을 파드에 주입(설정)한다.
      envFrom:
        - configMapRef:
            name: app-config # ConfigMap 정의 YAML 파일의 metadata 이름을 적는다.
```

이후 `kubectl create -f pod-definition.yaml` 명령어를 통해 파드를 생성하면, 컨피그맵을 통해 환경변수 설정이 주입된 파드가 실행될 것이다.

이러한 방식 외에도 컨피그맵을 주입하는 방식은 아래와 같이 몇 가지 더 있다.

### 파드에 컨피그맵을 주입하는 여러 방법

```yaml
# 위에서 살펴본, 기본적인 방식
envFrom:
  - configMapRef:
      name: app-config # ConfigMap 정의 YAML 파일의 metadata 이름을 적는다.
```
```yaml
# 하나의 키 값만 가져오는 경우
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config # ConfigMap 정의 YAML 파일의 metadata 이름을 적는다.
        key: APP_COLOR # 가져오고자 하는 키 값을 입력한다.
```
```yaml
# 볼륨에서 전체 데이터를 파일로 가져오는 경우
volumes:
- name: app-config-volume
  configMap:
    name: app-config # ConfigMap 정의 YAML 파일의 metadata 이름을 적는다.
```

---
# 시크릿(Secret)
지금까지 쿠버네티스에서 컨테이너의 환경변수를 관리하기 위해 컨피그맵을 생성하고 이를 파드에 주입하는 것을 살펴보았다.
그런데, 환경변수에는 사용자의 비밀번호와 같이 민감한 정보들이 있을 수 있다. 예를 들어보자.

## 컨피그맵의 한계, 그리고 시크릿의 필요성
```python
# app.py

import os
from flask import Flask

app == Flask(__name__)

@app.route("/")
def main():

    mysql.connector.connect(host='mysql', database='mysql', user='root', password='123')

    return render_template('hello.html', color=fetchcolor())

if __name__=="__main__":
    app.run(host="0.0.0.0", port="8080")
```
위와 같은 app.py 파일이 있다고 해보자. mysql 데이터베이스에 접근하는 코드를 살펴보면, `user=’root’`, `password=’123’` 같은 민감한 정보들이 있다.
이러한 변수들을 단순히 컨피그맵을 통해서 표현하면 다음과 같이 매우 노골적으로 민감한 정보들이 표현된다.

```yaml
# config-map.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: passwrd
```

이러한 문제를 해결하기 위해서 `시크릿(Secret)`이 활용된다.
먼저, 시크릿의 생김새부터 간단히 살펴보면 다음과 같이 사람이 알아볼 수 없는 `hashed format`으로 값들이 표현되어 있는 것을 알 수 있다.

```yaml
# Secret

DB_Host: bXlzcWw=
DB_User: cm9vdA==
DB_Password: cGFzd3JK
```

ConfigMap과 마찬가지로 Secret 또한 다음 두 단계를 거친다.

- 시크릿 생성하기
- 파드에 시크릿 주입하기

## 시크릿 생성하기
컨피그맵과 마찬가지로 `명령형(imperative) 방식`과 `선언형(declarative)방식`으로 시크릿을 생성할 수 있다.

### 시크릿 생성하기 : 명령형(imperative) 방식

- `kubectl create secret generic {시크릿 이름} --from-literal={키}={값}`
  - 예) `kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=passwrd`

컨피그맵과 마찬가지로, 위와 같이 `--from-literal` 옵션을 사용하여 시크릿에 환경변수를 키-값 형태로 전달할 수 있다. 또한, `--from-literal` 옵션을 여러 번 사용하여 여러 개의 요소들을 한번에 입력할 수도 있다.


- `kubectl create secret generic {시크릿 이름} --from-file={파일의 경로}`
  - 예) `kubectl create secret generic app-secret --from-file=app_secret.properties`

컨피그맵과 마찬가지로, 위와 같이 파일의 경로를 지정해주는 방식도 가능하다.


### 시크릿 생성하기 : 선언형(declarative) 방식

```yaml
# secret-data.yaml

apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw= # 참고 : echo -n 'mysql' | base64
  DB_User: cm9vdA== # 참고 : echo -n 'root' | base64
  DB_Password: cGFzd3Jk # 참고 : echo -n 'passwrd' | base64
```

`kubectl get secrets` 또는 `kubectl describe secrets` 명령어를 통해 생성된 시크릿을 확인할 수 있다.
`kubectl get secret app-secret -o yaml` 명령어를 통해 YAML 형식으로도 확인할 수 있다.


## 파드에 시크릿 주입하기
위의 방식대로 시크릿을 생성했다고 하면, 생성된 시크릿을 파드에 주입해야 한다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080

      # 아래 envFrom 부분을 통해 생성된 Secret을 파드에 주입(설정)한다.
      envFrom:
        - secretMapRef:
            name: app-secret # Secret 정의 YAML 파일에 설정한 metadata 이름을 적는다.
```
이후 `kubectl create -f pod-definition.yaml` 명령어를 통해 파드를 생성하면, 시크릿을 통해 환경변수 설정이 주입된 파드가 실행될 것이다.

이러한 방식 외에도 시크릿을 주입하는 방식은 아래와 같이 몇 가지 더 있다.
### 파드에 시크릿을 주입하는 여러 방법
```yaml
# 위에서 살펴본, 기본적인 방법
envFrom:
  - secretMapRef:
      name: app-secret # Secret 정의 YAML 파일에 설정한 metadata 이름을 적는다.
```
```yaml
# 또는 하나의 키 값만 가져오는 경우
env:
  - name: DB_Password
    valueFrom:
      SecretKeyRef:
        name: app-secret # Secret 정의 YAML 파일에 설정한 metadata 이름을 적는다.
        key: DB_Password # 가져오고자 하는 키 값을 입력한다.

```
```yaml
# 또는 볼륨에서 전체 데이터를 파일로 가져오는 경우
volumes:
- name: app-secret-volume
  secret:
    name: app-secret # Secret 정의 YAML 파일에 설정한 metadata 이름을 적는다.
```

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `쿠버네티스 공식 documentation` : [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)<br>
[4] `쿠버네티스 공식 documentation` : [Secrets](https://kubernetes.io/ko/docs/concepts/configuration/secret/)

