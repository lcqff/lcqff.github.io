---
layout: post
title: 인프라 스터디 (1) Docker로 프론트 배포하기
description: "Docker 스터디를 통해 학습한 프론트 배포 방법은 정리한다."
author: pam
categories: Infra
tags: Backend Infra Docker
image: https://github.com/lcqff/lcqff.github.io/assets/71930280/d005c395-b105-4055-a7af-08330a63fcec
toc: true
---
## 개요

두레 인프라 인수인계를 받기 위해 인프라 스터디에 참가했다🫡

첫번째 과제는 Keeper 홈페이지의 프론트 코드를 docker를 사용하여 배포하는 것이었다. 과제 해결을 위해 공부한 내용을 정리했다.


## Docker?
  <div class="captionedImg">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/33ab6cf3-b113-45ef-bc36-1bc62b991403">
    <p >https://smartstore.naver.com/dsticker/products/5945531926</p>
  </div>


### Environment Disparity

개발 환경의 차이로 발생하는 문제를 Environment Disparity라 한다. (윈도우에서는 동작하는 코드가 리눅스에도 동작하지 않음)

**Docker**를 통해 다른 머신에 동일한 환경을 구축하는 것으로 우리는 이러한 Environment Disparity를 해결할 수 있다.

### 저한테는 가상머신이 있는데요? (VM vs Docker)

사람에 따라 가상머신이 더 친숙한 사람도 있을 거다. 나도 학교 수업에서 Docker 대신 Vmware이나 VirtualBox를 사용해 그램 노트북으로 리눅스를 돌린 기억이 있다. 가상머신만 켰다 하면 노트북이 엄청나게 느려졌어서 그닥 좋은 기억은 아니다.

![spongebob meme](https://github.com/lcqff/lcqff.github.io/assets/71930280/5e4b6322-a845-4295-8650-b4b8338cb60c)

Vmware, Virtual Box를 포함한 Virtual Machine과 Docker의 가상화 방식에는 큰 차이가 하나 있는데, 바로 **Guest OS**의 유무이다. 

가상머신은 **호스트 OS 위에 가상화 소프트웨어를 이용해 여러개의 OS를 구동하는 방식**으로 이루어진다. 이러한 게스트 OS들은 하이퍼바이저에 의해 호스트 OS로부터 완벽하게 독립된다. 

{: .box-yello}
- 전가상화: 하이퍼바이저가 가상화된 하드웨어를 게스트 OS에게 제공하는 방식으로 독립시킨다.
- 반가상화: 하이퍼바이저를 통해서만 호스트 OS에 접근할 수 있도록 하는 방식으로 독립시킨다.

![vm vs docker](https://github.com/lcqff/lcqff.github.io/assets/71930280/d005c395-b105-4055-a7af-08330a63fcec)

그러나 도커 컨테이너를 사용한 가상화에서는 Host OS가 존재하지 않는다. VM처럼 물리적으로 공간을 격리하는 것이 아니라 **프로세스를 격리**하기 때문이다. 이로써 각 호스트 OS별 제공해야했던 자원이 줄어들어 더 많은 애플리케이션을 실행할 수 있으며, 호스트 OS를 실행시킬 필요가 없기 때문에 애플리케이션 실행 시간이 빠르다. 

<br>

![docker on vm](https://github.com/lcqff/lcqff.github.io/assets/71930280/b0b457c4-3b1f-4125-bdf1-8572d586cee7)

단 docker engine은 리눅스에서만 구동되기 때문에 mac이나 window와 같은 운영체제에서 사용시 vm으로 리눅스를 띄우고 그 위에 docker를 실행시켜야한다.

docker desktop 사용시 vm을 생성하는 복잡한 과정을 해결해주기 때문에 편하다.

### 이미지(image)

VM과 도커 컨테이너는 모두 이미지(image)로부터 생성된다. 이미지는 가상화된 환경의 청사진이며, 각 애플리케이션을 실행하는 데 필요한 독립적인 환경과 시스템 리소스를 지정한다. 

![docker image](https://github.com/lcqff/lcqff.github.io/assets/71930280/2e869d0e-583e-4fe8-8741-32d53dba6466)

단, VM 이미지는 전체 운영 체제, 라이브러리, 응용 프로그램 및 설정을 포함하는 단일 파일로 구성돼있는 반면 도커 이미지는 계층(**layer)**을 가진다. 

도커 이미지의 각 계층은 불변(Read only)이며 불변 레이어를 추가하는 것으로 필요 환경에 맞출 수 있다. (예를 들어, ubuntu 이미지 위에 레이어를 하나 추가하는 것으로 nginx 이미지가 된다.) 이러한 이미지의 불변성을 통해 개발자는 안정적이고 균일한 조건에서 소프트웨어를 테스트하고 실험할 수 있게 된다.

이러한 도커 이미지를 생성하기 위해 **Dockerfile**을 사용한다.

### 컨테이너(Container)

![docker process](https://github.com/lcqff/lcqff.github.io/assets/71930280/004357e2-53ec-43f5-9260-cea8a19135ee)

![container](https://github.com/lcqff/lcqff.github.io/assets/71930280/4c70a02b-9b81-4398-b3bb-a3150aa4e83d)

컨테이너는 이미지를 통해 생성되는 이미지의 인스턴스로, 이미지를 실행할 때 생성되는 가상 환경이다. 이미지는 실행될 수 없으나 이미지 위에 Container layer를 추가하는 것으로 호스트와 다른 컨테이너로부터 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 가상환경이 생성된다.


## 프론트 배포하기

인프라 스터디의 첫주차 과제는 Keeper 홈페이지의 프론트를 배포하는 것이었다.

### Dockfile 작성 & 빌드

```docker
#가져올 이미지 정의(from dockerhub)
#베이스 이미지 지정
FROM node:12

#작업 디렉토리 변경(경로 설정)
WORKDIR /app

#파일 또는 디렉토리 추가
COPY package*.json ./
# 호스트 운영체제의 현재 디렉토리 상에 있는 ~파일들을 복사해서 이미지의 현재 디렉토리에 복사해줌

#명령어 실행
RUN npm install
# 현재 디렉토리에 있는 package.json 파일들 읽어서 설치

EXPOSE 3000
#컨테이너에서 사용할 포트 정보

#컨테이너 생성시 실행할 명령어
CMD node app.js

#docker build -t nodejs-server .
```

이미지를 빌드하기 위해 프로젝트의 루트 위치에 Dockerfile을 작성했다. 
`docker build -t nodejs-server .` 명령을 통해 nodejs-server라는 이름으로 Dockerfile을 빌드하여 이미지를 생성할 수 있다.

<br>

{: .box-error}
ERROR: failed to solve: process "/bin/sh -c npm install" did not complete successfully: exit code: 1

dockerfile 빌드 과정에서 위와 같은 오류가 발생했다. 위 오류는 로컬의 package-lock.json을 제거하고 다시 이미지를 빌드하니 해결되었다.

<br>

![image cli](https://github.com/lcqff/lcqff.github.io/assets/71930280/98e04a2a-08a6-4761-b9d1-c71efa303fd1)

이미지가 생성된 걸 확인할 수 있다.

### Docker-compse 작성

하나의 컨테이너가 여러 응용 프로그램을 포함할 수도 있으나 일반적으로 한 컨테이너는 하나의 프로세스만을 가진다. 컨테이너를 만드는 비용이 크지 않기 때문에 컨테이너를 여러개 만들어 관리하는 것이 더 효율적이고 설계 변경에도 용이하다. 키퍼 홈페이지도 한 서버에 프론트, 백, 데이터베이스, 레디스까지 해서 총 4개의 컨테이너가 떠있는 상태다.

여러 컨테이너를 실행시키기 위해 직접 CLI에 각 컨테이너마다 run 명령어를 작성할 수도 있겠지만 아무래도 번거롭고, 동작을 확인하기도 어렵다.

**도커 컴포즈**는 단일 서버에서 여러개의 컨테이너를 하나의 서비스로 정의해 컨테이너의 묶음으로 관리할 수 있는 작업 환경을 제공한다.

참고로 window나 mac은 docker 데스크탑 애플리케이션을 다운받으면 자동으로 적용된다. 

```docker
 docker run -d -p 3000:3000 --name react-app nodejs-server
```

위와 같은 기존의 도커 명령을 아래와 같은 docker-compose 파일로 작성할 수 있다.

<br>

```java
version: "2"

services:
  web:
    build: . # 현재 경로에 이미지 빌드
    ports: 
      - "3000:3000" #포트 포워딩
    image: nodejs-server
    volumes: 
      - ./src:/app/src:ro
      #호스트 머신의 현재 작업 디렉토리 내 src 폴더를 컨테이너 내부의 /app/src 경로에 읽기 전용으로 마운트
      #호스트 데이터를 컨테이너에게 공유
    networks:
      - testnet

networks:
  testnet:
    external: true 
```
그러나 위 코드는 제대로 작동하지 않았다😅 원인은 차후 설명한다.


#### volume

도커는 각 컨테이너마다 독자적인 저장소을 가지며, 해당 데이터는 다른 컨테이너에 공유되지 않는다.

그러나 사용자는 항상 같은 서버에 접속하는 보장이 없으며 만일 사용자가 다른 서버에 접속하게 된다면 이전에 저장한 데이터를 조회하지 못하게 된다. 

따라서 사용자 상태 유지를 위해 모든 컨테이너는 하나의 저장공간을 공유해야한다.이를 위해 Docker에서는 **컨테이너가 데이터를 유지하고 공유할 수 있도록하는 저장공간인 Volume**을 제공해준다.

(사용자 입력 데이터와 관련됬다기 보단 호스트 데이터를 컨테이너에 공유하는데 사용되는 거 같다.)

볼륨은 기본적으로 호스트 디렉토리의 **`/var/lib/docker/volumes`** 경로에 저장된다. 그러나 docker는 리눅스 기반의 프로그램이고, mac에서 사용하는 경우 vm 위에 docker가 띄워지기 때문에 mac에는 해당 경로가 존재하지 않는다. 

따라서 우리는 호스트의 저장 공간을 컨테이너의 저장공간에 **마운트** 할 필요가 있다.


<div class="callout">
  <div>💡</div>
  <div>
    <strong>마운트</strong><br/>
    파일 시스템의 특정 위치를 다른 위치에 연결하는 프로세스<br/>
    Docker에서는 호스트와 컨테이너 간에 데이터를 공유하기 위해 호스트의 파일 시스템 경로를 컨테이너의 특정 경로에 연결하는 것을 의미한다.
  </div>
</div>

마운트는 docker-compose 파일에서 아래와 같은 형태로 이루어진다. 이를 통해 호스트와 컨테이너 간에 데이터를 주고받거나 공유할 수 있다.

<br>

```java
    volumes:
      - /호스트/저장/경로:/컨테이너/내부/경로
```


#### Network

컨테이너는 서로 독립된 환경에서 동작하기 때문에 서로 통신할 수 없다. 따라서 각 컨테이너간 통신하기 위해서는 컨테이너들을 하나의 네트워크로 연결해주는 작업이 필요하다. 

```java
networks:
  testnet:
    external: true 
```


<br>

자, 이제 docker-compose 파일을 작성 완료했다면 `docker-compose up` 명령어를 통해 컨테이너를 실행해보자.

## 오류발생

![error](https://github.com/lcqff/lcqff.github.io/assets/71930280/092cfe18-f3ea-45ab-97e4-d0ba7833f0b6)

컨테이너가 실행되었으나 **그와 동시에 바로 종료**되었다. log를 확인해보니 react를 import해오는 부분에서 `Cannot use import statement outside a module` 오류가 발생한다.

<br>

오류를 찾아보니 Typescript와 관련된 문제였다. Keeper 홈페이지는 Typescript를 통해 개발되었기 때문에 <red>Typescript 언어를 Javascript로 컴파일해주는 과정이 필요</red>했다.

```java
RUN npm install
# 현재 디렉토리에 있는 package.json 파일들 읽어서 설치

RUN npm install -g typescript
# typescript 모듈 설치.  

COPY ./ ./
# 현재 폴더에 있는 모든 파일들을 복사하여 이미지 디렉토리에 복사합니다. 
# 이 과정에서 자신이 작성한 코드등 리소스들이 복사됩니다.  

RUN tsc
# typescript로 작성된 node이므로 javascript 파일로 컴파일 해줍니다.  
```

이는 위 명령을 추가하여 해결할 수 있었다.

<br>

{: .box-error}
[7/7] RUN tsc -p .:
0.805 /usr/local/lib/node_modules/typescript/lib/tsc.js:93
0.805   for (let i = startIndex ?? 0; i < array.length; i++) {
0.805                            ^
0.805
0.805 SyntaxError: Unexpected token '?'


tsc 과정에서 위의 오류가 발생했는데, 이는 내가 베이스 이미지로 가져온 node 이미지의 버전 문제였다. ?? 문법을 이해할수 있는 18버전 노드로 이미지를 교체했다.

### 최종 docker-compose.yml

```java
FROM node:18

WORKDIR /app
#작업 디렉토리 변경(경로 설정)

COPY package*.json ./
# 호스트 운영체제의 현재 디렉토리 상에 있는 ~파일들을 복사해서 이미지의 현재 디렉토리에 복사해줌

RUN npm install
# 현재 디렉토리에 있는 package.json 파일들 읽어서 설치

RUN npm install -g typescript
# typescript 모듈 설치.  

COPY ./ ./
# 현재 폴더에 있는 모든 파일들을 복사하여 이미지 디렉토리에 복사합니다. 
# 이 과정에서 자신이 작성한 코드등 리소스들이 복사됩니다.  

RUN tsc
# typescript로 작성된 node이므로 javascript 파일로 컴파일 해줍니다.  

EXPOSE 3000
#컨테이너에서 사용할 포트 정보

CMD ["npm", "run", "start"] 
```

<img width="1461" alt="image" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/7902a304-0763-4024-90a9-a9d6034b8e28">

성공~

<br>

---

인프라를 공부하니 확실히 전보다 훨씬 폭넓은 수준의 지식을 획득할 수 있는 거 같아 좋았다😀



<br><br>

참고

[프론트엔드 개발자를 위한 Docker로 React 개발 및 배포하기](https://velog.io/@oneook/Docker로-React-개발-및-배포하기)