---
layout: post
title: "23.11.22 블로그 업데이트4"
author: pam
categories: 블로그_업데이트
sidebar: []
---



## 추가 기능

<br/>
<br/>

**1. NEW 태그 삭제**
<img alt="new 태그가 달린 모습" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/e59336ed-5fd5-4f78-806e-eed94d509918">

- 블로그 테마에서 제공하는 기능 중에 포스트 타이틀 옆에 new 태그를 달아주는 기능이 있었다.
- 그러나 최신 글에 new를 다는 게 아니라... 읽어보지 않은 글에 new가 달리는 로직이라 블로그에 처음 접속하게 되면 모든 글에 new가 뜨게 된다😂
- 나중에 최신 글에만 new가 뜨도록 고칠수 있다면 기능을 되살리고 싶다...
- `_layout.scss`에서 코드를 세 개 정도 주석 처리함.. 이후에 new 검색해서 찾아보기

**2. box css 또 수정**
<div>
    <img alt="수정 전" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/77a9345f-cb1f-4241-bb7f-3544e156893f">
    <img alt="수정 후" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/a64315db-a8c5-4e30-bdc0-617f87bfdb19">
</div>
{: .imageRow}

- 또 수정했다.. 이전에 box 레이아웃은 list가 삽입되면 위와 같이 box 옆에 마진이 생기는 문제가 있었다.
- 마진 없애고 패딩을 더 늘리는 식으로 수정했다.


**3. 배너 사이즈, favicon 변경**
<div>
    <img alt="변경 전" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/e9b752b7-3bac-46d3-9dce-c93bd4d68201">
    <img alt="변경 후" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/e24e6a2d-2402-4244-98b6-f719969d7682">
</div>
{: .imageRow}

- 배너의 높이가 너무 큰거 같아서 줄였다  
- 하는 김에 favicon도 변경된 걸로 갈아치웠음  
- `_config.yml` 수정