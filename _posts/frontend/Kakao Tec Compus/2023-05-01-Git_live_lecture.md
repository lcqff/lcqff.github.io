---
layout: post
title: "라이브 특강-Git 후기"
author: 사탄
categories: KakaoTecCampus
banner:
  image: https://user-images.githubusercontent.com/71930280/235451440-2f238ca6-f01a-4a8d-a1ab-bf7426239a00.png
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: 카태캠 kakaoTecCampus 후기
---

Git 라이브 특강은 Zoom을 통해 4/23,5/1 이론과 실습 총 두 차례에 나눠 이루어졌다.

### 이론

첫번째 강의에서는 강사님이 git flow에 대한 이론을 가르쳐주셨다.
나는 실제 개발에서는 Source Tree라는 GUI Repository를 사용했는데 강사님이 말씀하시길, GUI Repository를 사용하기 보다는 CLI를 사용한 git 사용에 익숙해지는 편이 좋다고 말씀하셨다.
CLI를 사용해 복잡한 git 작업을 해본 건 아니었지만 간단한 정도라면 어느정도 경험이 있기에 큰 문제 없이 수업을 따라갈 수 있었다. 특히, 나는 보통 CLI를 사용할 때 git commit -m "내용"을 사용해서 commit을 작성했는데 강사님이 git commit 명령을 통해 vim으로 commit을 작성하는 법을 알려주신게 큰 꿀팁이 되었다.

두번째에는 조원과 함께 직접 실습을 하는 방향으로 이루어졌다. 2번째 강의 전에는 실습 전 예습해야 하는 내용을 영상으로 올려주셨는데 나는 예습을 하라는게 1번째 실시간 강의 내용을 예습해오라는 뜻인 줄 알고 안보고 있다가 친구가 카톡을 줘서 2번째 강의 시작 30분 전에야 겨우 확인했다--; 예습해야하는 강의는 45분 짜리라서 당연히 다 보진 못했지만... 다행히 실습 내용이 그렇게 어렵진 않았다.

### 실습

실습은 팀원들과 함께 진행됐다.

[BullsAndCows](https://www.mathsisfun.com/games/bulls-and-cows.html) 또는 [Monty Hall Simulation](https://www.mathwarehouse.com/monty-hall-simulation-online/) 프로그램을 git울 사용해 팀원들과 협동하여 구현하는 내용이었다.

우리 팀은 BullsAndCows를 선택하였다. Monty Hall Sibulation에 비해 이해가 쉬웠기 때문이었다...ㅎㅎ
팀원 4명이 BullsAndCows 프로그램을 크게 main, play, ckend, ckInput으로 나누어 맡고 팀장님은 올라온 PR들의 코드리뷰를 맡기로 했다.

github에서 Organization을 만든 뒤 각자 역할에 맞게 Issue를 생성하고, CLI를 사용하여 이론 강의 때 배운 대로 git clone, 브랜치 생성, add, commit, push해 PR을 올리는 간단한 실습이었다.

사실 그렇게 어려운 내용의 실습은 아니었다. 이전에 많이 해봤던 거기도 하고... 하지만 이런 내용으로 실습이 진행될 거라(팀원들간 실습이 있을거라고 하시긴 했다😅) 예상하지 못해 제일 초반, 코드를 작성하기도 전에 꽤 버벅인 부분이 적지않아 있다. 특히 역할을 나누고 이슈를 생성하는 부분에서 버벅였던 거 같다. + 내가 fork를 안하고 push를 해버리는 바람에 뜨는 권한없음 오류 때문에 당황해서 헤매느라 시간을 썼고, 여기에 시간을 쓰느라 진행이 막힌 다른 팀원을 돕지 못했다😔

주어진 시간은 꽤 길었으나 위와 같은 이유로 종료 시간까지 테스트를 못해본 코드는 오류가 나서 돌아가지 않았다.... 슬프다!

실습은 강의 종료 15분 전에 끝났고, 남은 시간동안은 강사님이 push 시 충돌이 나는 케이스를 간단하게 시현해주셨다. github에 실습한 내용들은 강사님이 이번주 내로 피드백 해주시겠다고 한다.
