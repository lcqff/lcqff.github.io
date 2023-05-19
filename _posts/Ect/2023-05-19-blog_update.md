---
layout: post
title: "23.5.19 블로그 업데이트"
author: 불한당
categories: 블로그_업데이트
sidebar: []
---

키퍼 발표가 2시간이 걸려서… (오지말걸) 이번에 키글챌 스터디(?)에도 참여한 김에 키퍼 발표를 들으며 블로그에 몇몇 기능을 추가했다.

## 추가 기능

1.**게시글 내부 사이드바 추가**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/f8927b65-be82-46e6-b08b-429e8a2d4b66)

- 기존 기능이었는데 사용을 안하고 있었다.
- 사용하고 있는 테마의 예시 블로그를 확인하다 발견
  - 게시글 마크다운 파일 상단에 `sidebar: []` 삽입으로 제거 가능

<br/>
<br/>

2.**대망의 댓글 기능 추가**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/d70565b1-4bed-4599-9b94-38d3fc624933)

- 매우 간단하게 블로그에 댓글 기능을 추가할 수 있었다…
- 위에 있는 responses 기능도 disqus에서 함께 제공해준다. (off 가능)
- 심지어 블로그 테마 제작자 분이 이미 해당 기능을 위한 코드를 주석 처리해놓았다😅 코드를 더 잘 살펴보는 것이 좋겠다.
  - **disqus**사용, `config.yml` 파일 내 disqus 설정 존재
  - disqus사이트에서 설정 변경 가능. [사이트](https://https-lcqff-github-io.disqus.com/admin/settings/general/)
- 비회원으로 댓글 작성이 가능하지만 그래도 이메일 주소는 작성해야한다.
- 이후 좀 더 살펴봐야겠다.

<br/>
<br/>

3.**블로그 배너 추가, 사이드 헤딩 변경**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/3cc6598a-1910-4943-b1aa-e59c844338f1)

- 현재 디폴트 이미지 적용 중… 이후 변경예정
- `_data/defaults.yml`
