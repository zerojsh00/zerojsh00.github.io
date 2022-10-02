---
title: (K8S) 도커의 스토리지(Storage in Docker)
author: simon sanghyeon
date: 2022-09-03
categories: [Kubernetes]
tags: [Kubernetes, K8S, Storage]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
이번에는 `file system`과 `docker storage driver`에 대해서 간략히 살펴보고자 한다.
즉, 도커가 데이터를 어떻게 저장하는지, 그리고 컨테이너의 파일 시스템을 어떻게 관리하는지를 살펴본다.

도커가 로컬 파일 시스템에서 데이터를 어떻게 저장하는지부터 살펴보자.
도커를 설치하면, 도커는 `/var/lib/docker` 폴더 경로를 생성한다. 이 경로 밑에는 `aufs`, `containers`, `image`, `volume` 등 다양한 폴더들이 생성된다. 이곳이 default로 도커가 데이터를 저장하는 경로가 된다.
예컨대, 컨테이너와 관련된 모든 파일들은 containers 폴더, 이미지와 관련된 모든 파일들은 image 폴더에, 그리고 도커 컨테이너에 의해서 만들어진 볼륨들은 volume 폴더에 저장된다.

---
# Recap : 도커의 레이어 구조
그런데, 도커는 어떻게 이미지와 컨테이너의 파일들을 저장할까? 이를 알기 위해서는 `도커의 레이어 구조`를 알아야 한다.

아래와 같은 Dockerfile이 있다고 해보자.

```docker
FROM Ubuntu

RUN apt-get update && apt-get -y install python

RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

`docekr build Dockerfile -t simon/my-custom-app`

또 위와 매우 유사한, 아래와 같이 Dockerfile2가 있다고 해보자.

```docker
FROM Ubuntu

RUN apt-get update && apt-get -y install python

RUN pip install flask flask-mysql

COPY app2.py /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app2.py flask run
```

`docekr build Dockerfile2 -t simon/my-custom-app-2`

Dockerfile2는 Dockerfile에서 일부만 변경되었다. 이 경우 Dockerfile2를 위해서 120MB가 되는 우분투 이미지(첫 줄)와 300MB 정도 되는 apt package(두 번째 줄) 등을 모두 다시 받아야 한다면 매우 비효율적이다.

따라서 도커는 `레이어 구조`를 사용한다. 즉, Dockerfile과 Dockerfile2 모두 세 번째 줄까지는 동일한 내용이므로, 캐시로부터 동일한 레이어로 공유한다. 그리고 Dockerfile2에서 변경 사항이 생긴 네 번째 줄과 마지막 줄만 새롭게 레이어로 생성한다. 이러한 방식으로 도커는 이미지를 더욱 빨리 빌드할 수 있으며, 디스크 공간도 효율적으로 사용할 수 있는 것이다. 이를 그림으로 나타내면 아래와 같다.

![fig01](/assets/img/2022-09-03-Storage in Docker/fig01.png){: width="500" height="500"}

## 이미지 레이어(Image Layer)와 컨테이너 레이어(Container Layer)
다시 Dockerfile을 살펴보자. `docekr build Dockerfile -t simon/my-custom-app` 명령어를 통해서 Dockerfile을 빌드하면, `이미지 레이어(Image Layer)`라는 레이어가 다음과 같이 생성된다. 이 레이어들은 모두 `read only` 레이어이므로, 변경될 수 없으며, 부득이 변경하고자 한다면 Dockerfile을 변경 후 다시 빌드하는 수밖에 없다.

- **Layer 5** : Update Entrypoint with “flask” command
- **Layer 4** : Source code
- **Layer 3** : Changes in pip package
- **Layer 2** : Changes in apt package
- **Layer 1** : Base Ubuntu Layer

이후, `docker run simon/my-custom-app` 명령으로 이 Dockerfile을 실행(run)하면 image layer 위에 추가로 `컨테이너 레이어(Container Layer)`라는 레이어가 하나 더 생긴다. 이는 `read & write` 레이어로, write 기능이 있다. 즉, 로그 데이터라든지, temporary 파일이라든지, 혹은 사용자들에 의해서 변경되는 내용들 등 컨테이너에 의해서 생성되는 데이터들이 저장되는 데 사용되는 레이어다.

- **Layer 6**: Container Layer

이러한 컨테이너 레이어는 컨테이너가 살아있을 때에만 존재하며, 컨테이너 실행이 중단되면 해당 레이어와 이곳에 저장되어 있던 변경 사항 데이터들도 사라진다.

---
# Copy-on-Wirte 메커니즘
![fig02](/assets/img/2022-09-03-Storage in Docker/fig02.png){: width="500" height="500"}

위와 같은 상황에서 read only인 이미지 레이어에 있는 app.py 파일에 변경 사항이 생기면 어떻게 해야 할까? 동일한 이미지 레이어는 해당 레이어를 사용하는 여러 컨테이너들에서 공유되기 때문에, read only인 이미지 레이어의 app.py는 변경이 불가능할까?

`Copy-on-Write(CoW) 메커니즘`을 사용하면 변경할 수 있다. CoW 메커니즘은 read only 레이어에 있는 파일에 변경 사항이 생길 때 이를 처리하는 가장 쉬운 방법이다.

![fig03](/assets/img/2022-09-03-Storage in Docker/fig03.png){: width="500" height="500"}

위와 같이, 우선적으로 read only인 이미지 레이어에 있는 파일을 write가 가능한 컨테이너 레이어에 복사한다. 그 후, write가 가능한 레이어에 복사된 파일에 대해서 변경 사항을 적용한다. 즉, 이미지 레이어에 있는 파일 자체가 변경되는 것은 아닌 셈이다. 이때 컨테이너는 변경 사항이 적용된 레이어의 파일만 볼 수 있을 뿐, 이미지 레이어에 존재하는 파일은 볼 수 없다.

그러나 이러한 방식에는 문제가 있다. 컨테이너를 삭제하면, 컨테이너 레이어도 함께 삭제되기 때문이다. 즉, 변경 사항이 반영된 app.py 파일도 함께 사라지게 된다. 따라서 영구적으로 데이터를 관리하는 방식이 필요하다.

---
# CoW 메커니즘의 한계를 보완하는 Volume Mount

CoW의 문제를 해결하는 방법으로 `volume mount`를 통해 컨테이너에서 persistent volume을 사용할 수 있다.

우선, `docker volume create {볼륨_이름}` 명령을 통해서 볼륨을 생성한다. 이는 `/var/lib/docker/volumes/{볼륨_이름}` 경로를 만들어 낸다. 이후, `docker run -v {볼륨_이름}:{컨테이너_내의_경로} {이미지_이름}`명령으로 이미지를 실행시키면, persistent volume이 마운트된다.

예를 들어, mysql 이미지를 data_volume이라는 이름의 볼륨에 마운트 하고자 한다면, `docker volume create data_volume` 명령으로 data_volume을 생성한 후, 컨테이너 내에서 mysql의 데이터가 저장되는 default 위치인 `/var/lib/mysql` 경로를 입력하여 `docker run -v data_volume:/var/lib/mysql mysql` 명령으로 mysql 이미지를 실행한다. 이를 도식으로 표현하면 다음과 같다.

![fig04](/assets/img/2022-09-03-Storage in Docker/fig04.png){: width="500" height="500"}

이처럼 volume mount를 활용하면 컨테이너를 삭제해도 데이터가 호스트에 보존될 수 있다.

한편, `docker volume create {볼륨_이름}` 명령으로 볼륨을 생성하지 않고서, `docker run -v` 명령으로 volume mount를 적용한다면 어떻게 될까? 이 경우, 도커에 의해 `-v` 옵션 이후에 입력한 볼륨 이름으로 볼륨이 자동으로 생성된 후 마운트 된다. 정리하자면, volume mount를 통해 도커의 볼륨 디렉토리와 컨테이너를 마운트 할 수 있는 것이다.

---
# Bind Mount

만약, 도커 호스트 내 어떤 곳에 이미 데이터가 존재한다면, 해당 데이터의 경로를 컨테이너에 `bind mount` 할 수 있다. 이 경우, `docker run -v {데이터가 존재하는 절대 경로}:{컨테이너_내의_경로} {이미지 이름}` 명령을 실행하여 bind mount 할 수 있다. 정리하자면, bind mount를 통해 도커 호스트 내 어떤 경로든 컨테이너와 마운트 할 수 있는 것이다.

---
# Storage Drivers

지금까지 살펴보았듯, 컨테이너를 실행할 때 write가 가능한 컨테이너 레이어를 만들고, 이미지 레이어로부터 파일을 복사하는 등, 스토리지와 관련한 일련의 과정을 수행하는 주체를 `storage driver`라고 부른다. storage driver는 `AUFS`, `ZFS`, `BTRFS`, `Device Mapper`, `Overlay`, `Overlay2` 등이 있으며, OS에 따라서 어떤 storage driver를 사용할지가 자동으로 결정된다. 예를 들어, 우분투의 경우 default로 AUFS를 사용한다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
