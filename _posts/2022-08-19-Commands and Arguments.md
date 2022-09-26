---
title: (K8S) 도커와 쿠버네티스의 매개변수 받는 방법
author: simon sanghyeon
date: 2022-08-19
categories: [Kubernetes]
tags: [Kubernetes, K8S, Docker, Argument, Command]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# [Docker] 간단히 살펴보는 Dockerfile의 CMD
## 프로세스가 종료되면 함께 종료되는 컨테이너
![fig01](/assets/img/2022-08-19-Commands and Arguments/fig01.png)
docker를 이용해서 우분투 이미지를 실행하면, 위와 같이 곧바로 `exit(종료)` 상태로 변경되어 버린다. 왜 그런 것일까?

컨테이너는 운영체제를 호스트 하지는 않고, 단지 웹 서버나, 데이터베이스와 같은 특정 프로세스들을 실행할 수 있게 한다. 또한 이러한 **내부 프로세스들이 살아있는 동안에만 컨테이너도 살아있기 때문에, 내부 프로세스들이 종료되면 컨테이너도 종료된다.**

## 컨테이너가 생성될 때 실행될 명령을 정의하는 CMD
그렇다면, 컨테이너 내부에 있는 프로세스는 어디에서 실행되도록 정의되어 있을까? 정답은 `도커 파일(Dockerfile)`이다. 보통 도커 파일을 열어보면, 다음과 같이 생겼다.

![fig02](/assets/img/2022-08-19-Commands and Arguments/fig02.png)

nginx를 설치하고 실행하는 이 도커 파일을 보면, 마지막 CMD 명령어 부분에 컨테이너 내부에서 실행될 프로그램 명령이 적혀있는 것을 알 수 있다. 이것이 실행되면 컨테이너 내부에서 해당 명령에 대한 프로세스가 실행되는 것이다.

![fig03](/assets/img/2022-08-19-Commands and Arguments/fig03.png)

앞서 우분투를 실행하기 위해 사용했던 우분투의 도커 파일을 열어보면, `CMD` 명령어 부분이 `bash`라고 되어 있는 것을 알 수 있다. bash는 nginx 웹 서버나 mysql 데이터베이스 같은 프로그램이 아닌, shell 이다. shell은 터미널로부터 input을 기다리고 있다가, 터미널을 찾지 못하면 종료된다.

앞서 실행했듯이, 우분투 컨테이너를 run하면, 도커는 우분투 이미지로부터 컨테이너를 만들고 bash program을 실행한다. 그런데 도커는 기본적으로 컨테이너가 실행될 때 컨테이너에 터미널을 붙이지 않는다. 따라서 bash program이 터미널을 찾지 못해서 종료되는 것이다. (컨테이너가 생성될 때 만들어졌던 프로세스가 종료되면, 컨테이너도 종료되는 것 또한 유사한 원리다.)

---
# [Docker] ubuntu sleep 명령 예제를 통해 살펴보는 CMD와 ENTRYPOINT
## 도커 파일의 CMD

`docker run ubuntu sleep 5` 명령을 입력하면, 실행 즉시 5초 동안 sleep 한 후 종료되는 우분투 컨테이너가 실행된다. 위와 같은 명령어는 command line에 파라미터를 이용해서 단 한번만 단편적으로 실행되도록 한다.

이러한 sleep 명령을 다음과 같이 아예 도커 파일에 정의하면, 매번 이미지를 실행할 때마다 5초간 sleep 후 종료되는 컨테이너를 만들 수 있다.

```docker
FROM Ubuntu
CMD sleep 5

# 혹은 CMD ["sleep", "5"]
```

이러한 도커 파일을 `ubuntu-sleeper`라는 이름으로 이미지를 빌드하고 실행하면, 5초간 sleep 후 종료되는 우분투 컨테이너가 실행되는 것이다. 이처럼 도커 파일에 `CMD`를 활용하여 `sleep` 명령과 `sleep 시간`을 정의하면, `docker run ubuntu-sleeper sleep {원하는 시간}`과 같이
{원하는 시간}을 파라미터로 줌으로써 sleep 시간을 오버라이드한 명령을 수행할 수도 있다. **즉, 핵심은 docker run 명령 시 CMD에 정의한 스크립트가 오버라이드 될 수 있다는 것이다.**

## 도커 파일의 ENTRYPOINT
### 반드시 실행되어야 할 명령어를 지정한다.
그런데 `docker run ubuntu-sleeper sleep {원하는 시간}`처럼 `{원하는 시간}`을 오버라이드 할 수는 있지만, `sleep`이라는 CMD 명령어를 입력해야 하는 불편함이 있다.
이 경우, `ENTRYPOINT`를 이용할 수 있다. ENTRYPOINT는 CMD와 비슷한데, `docker run` **명령으로 컨테이너가 시작되는 시점에 ENTRYPOINT의 명령어가 반드시 실행된다.** 예제를 살펴보자.

```docker
FROM Ubuntu
ENTRYPOINT ["sleep"]
```

위와 같이 명령어를 입력한 후, 마찬가지로 ubuntu-sleeper라는 이름으로 이미지를 빌드한 후, `docker run ubuntu-sleeper {원하는 시간}` 명령을 실행하면, ENTRYPOINT에 정의된 `sleep` 명령어가 반드시 실행되므로, 숫자 `{원하는 시간}`만이 매개변수로 작용하여 해당 시간만큼 sleep하게 된다.

## CMD와 ENTRYPOINT의 조합
### ENTRYPOINT로 Command를 지정하고, 매개변수의 기본값은 CMD로 정의한다.

```docker
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

도커 파일을 구성할 때, ENTRYPOINT와 CMD를 함께 사용하면, 이미지를 실행할 때 ENTRYPOINT를 반드시 사용하고, default 값으로는 CMD에서 정의된 값이 자동으로 사용된다.
또한 `docker run ubuntu-sleeper {원하는 시간}` 명령을 실행하여 매개변수로 원하는 값을 입력함으로써 CMD에 정의되어 있는 default 값을 오버라이딩 할 수도 있다.

---
# [Kubernetes] 컨테이너의 매개변수를 받는 방법
앞서 살펴본 `ubuntu-sleeper` 예제를 쿠버네티스에서 배포하면 매개변수를 어떻게 처리해야 하는지 살펴보자.

## args를 활용하여 매개변수 받기
```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
```

ubuntu-sleeper 도커 파일을 빌드하여 이미지를 생성하면, 위와 같이 파드를 정의하는 YAML 파일을  만들어볼 수 있다. 그런데, 앞서 도커 컨테이너를 이용한 예제에서는 매개변수로 sleep 시간을 command line에 추가로 넣어줄 수 있었다. 파드 생성 시에는 어떻게 해야 할까?

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"] # 추가된 부분! 파드에서는 이와 같이 매개변수를 입력한다.
```
위 YAML을 보면, 마지막 줄에 args를 통해 매개변수를 입력할 수 있다. 도커 파일과 함께 보면 더욱 이해하기 쉽다.

```docker
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

YAML 파일의 args는 위와 같은 도커 파일의 CMD 부분을 오버라이드 하는 것이다. `kubectl create -f pod-definition.yaml` 명령을 입력하면 도커 파일의 CMD 부분이 오버라이드된 파드가 생성된다.

## 쿠버네티스에서 Dockerfile의 ENTRYPOINT 오버라이드 하기
만약, 도커 파일의 ENTRYPOINT를 오버라이드 하고 싶다면 어떻게 해야할까?
참고로 도커 명령어로는 `docker run --name ubuntu-sleeper --entrypoint sleep2.0 ubuntu-sleeper 10`과 같은 방식으로 ENTRYPOINT를 컨테이너 실행 시 오버라이드 할 수 있다.
그렇다면, 파드로는 어떻게 할까? 이는 파드 정의 YAML의 `command`에 대응된다.

```yaml
# pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"] # 도커 파일의 ENTRYPOINT는 파드 생성 시 'command'에 대응된다.
      args: ["10"]
```

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
