---
layout: post
title: Github Docker Migration (feat. CICD docker-compose command not found)
author: 팜
categories: DooRe
description: \[troubleShooting] DooRe의 CICD 과정엑서 갑작스럽게 발생한 에러 해결 과정
tags: DooRe troubleShooting CICD
sidebar:
---

## 0. 개요

**갑자기 CICD가 실패한다!**

이전까지 잘 작동했고, 수정한 사항이 없었기 때문에 갑작스럽게 느껴진 에러였다.

BDD가 거처를 디스코드로 옮김으로서 슬랙이 폐쇄되고, 그 과정에서 슬랙과 연동된 알림에 문제가 생겨 발생한 에러인가 처음에 생각했으나. 슬랙 알림이 문제라기엔 발생한 에러는 docker-compose쪽 문제….🤔

대체 원인이 무엇인지 알아보러 가자.

## 1. 문제

![image](https://github.com/user-attachments/assets/8e0c8dc9-4145-408d-ae28-eca2ec9bbaa2)

위와 같은 문제가 발생함을 확인했다. 

BDD 슬랙이 폐쇄되어, 슬랙에 알림을 전송하는 코드에서 오류가 발생하고 있음을 알 수 있었다. 

단순히 슬랙에 알림을 전송하는 코드를 주석 처리하면 해결될 문제라고 생각했으나,,,


```yml
      # - name: Notify Slack
      #   if: always()
      #   uses: 8398a7/action-slack@v3
      #   env:
      #     SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      #   with:
      #     status: ${{ job.status }}
      #     author_name: DOO_RE Devlopment Backend CICD
      #     fields: repo, commit, message, author, action, took
```
<cap>주석 처리된 코드</cap><br>

<br>


![image](https://github.com/user-attachments/assets/18643b22-da32-47c6-9d1f-a2dc5dc8d1f1)

`docker-compose: command not found`에러는 여전하다. 🤔
이는 docker-compose 명령을 찾지 못한다는 의미로, 원래 정상적으로 작동해왔던 코드이기에 이상함을 느꼈다. 

## 2. 고민

가장 먼저 고려할 수 있었던 것은 docker-compose 설치 코드를 추가하는 것이었다. 

그러나 (자세히 아는 부분은 아니지만) 우리는 배포 과정에서 docker hub를 쓰는 것도, 따로 self-hosted runner를 사용하는 것도 아니었기 때문에 **Github Ubuntu**를 CICD환경으로 사용해왔다. 그런데 갑자기 Docker Compose 설치가 되어있지 않다고 에러가 뜨는 것은 이상하다…🫤

## 3. 해결

> GitHub deprecated v1, and you need to change the command from, e.g., `docker-compose build` to `docker compose build` (remove the dash)


**Docker v1** 사용 문제였다!

 Github에서 더 이상 v1 Docker를 사용하지 않기로 업데이트 된 모양이다. 그래서 Github Ubuntu를 사용한 CICD 환경에서 V1 Docker 명령어를 인식하지 못해 에러가 발생했던 거 같다.

```yml
      - name: Start Containers
        run: docker-compose -p doo-re up -d
```
<cap>수정 전(V1 명령어 사용)</cap><br>

```yml
      - name: Start Containers
        run: docker compose -p doo-re up -d
```
<cap>수정 후(V2 명령어 사용)</cap><br>

V2 명령어로 CICD 스크립트를 수정하는 것으로 해결할 수 있었다. 

Docker V1 → V2 migration과 관련된 내용은 아래 링크에서 더 자세하게 확인할 수 있다. 

- [Error: docker-compose command not found · community · Discussion #116610](https://github.com/orgs/community/discussions/116610)
- [Migrate to Compose V2](https://docs.docker.com/compose/migrate/)

---

두레에서 더 이상 백엔드 작업은 진행하지 않기로 했다😥 (서버 이전까지만 진행하기로...)
두레를 통해 백엔드 개발을 처음 시작했고, 많은 애정을 쏟고, 열심히 일했던 프로젝트이지만 솔직히 말해서 현재 두레의 코드는 영 만족스럽지 않다.
그래서 코드를 개선하고 싶었지만... 나의 의지와는 관계없이 더 이상 두레를 통한 새로운 도전도, 두레의 코드 개선도 불가능할거 같다는 생각이 들었고, 이것이 큰 스트레스였다😅
새로운 도전을 할 수 있는 다른 프로젝트를 시작해보려한다. 
그래도 인프라 이슈가 생길 때마다 포스팅은 업데이트 될 예정이다.
