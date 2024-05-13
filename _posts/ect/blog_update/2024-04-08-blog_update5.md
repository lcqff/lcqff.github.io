---
layout: post
title: "24.04.08 블로그 업데이트5"
author: 팜
categories: 블로그_업데이트
tags: gitblog
---


## 추가 기능

<br/>
<br/>

**1. ImageRow width 제한**
<img width="995" alt="imgaeRow" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/e280f099-de37-46e0-987d-afc8c7052b97">

- 한 행에 이미지를 정렬하는 ImageRow를 대충 css로 만들어서 쓰고 있었는데 이미지가 3개 이상 정렬되면 body에서 튀어나가는 게 굉장히 거슬렸다.
- css가 아닌 js를 사용해 ImageRow 클래스 내에 있는 이미지의 width 총합이 body의 width와 동일하도록하는 로직을 추가했다.
- `_includes/views/article.html`내에 코드를 작성했다.

