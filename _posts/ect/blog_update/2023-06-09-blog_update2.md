---
layout: post
title: "23.6.9 블로그 업데이트2"
author: 퍄녀미
categories: 블로그_업데이트
sidebar: []
---

<style>
  .imageRow {
    display:flex;
    margin: 20px 0;
  }
    .captionedImg {
    display: grid;
    align-content: flex-end;
    margin: 0 20px;
    text-align:center;
    font-size: 12px;
    color:gray;
  }
</style>

## 추가 기능

<br/>
<br/>
**1. 박스 레이아웃 추가**

<div class="imageRow">
  <div>
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/bc58c870-e157-4b26-9d03-f593adeacfe4">
  </div>
  <div>
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/2ba6e79b-e6cf-4bfe-8e05-0d41b84794b3">
  </div>
</div>

- 테마 샘플 포스트엔 존재했지만 실제로는 적용되지 않던 박스 레이아웃을 추가했다.
- note 박스가 dark 모드에선 안보여서 하얀색 테두리 추가함
- `_sass/yat/_layout.scss` 파일 참고
- 사용은 test-markdown 게시물 참고

<br/>
<br/>
**2. 콜아웃 추가**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/e116e668-8167-4ae5-ab4a-c66c9efc8507)

- 콜아웃 레이아웃을 제작하였다.
- 디자인은 노션에 있는 콜아웃 뺏겨옴ㅎㅎ
- `_sass/yat/_layout.scss` 파일 참고
- 사용은 test-markdown 게시물 참고

<br/>
<br/>
**3. favicon 추가**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/8aa6092f-76b7-43e5-a8fc-8a858739008f)

- relative path가 아닌 absolute path 사용할 것 (안그럼 게시글 클릭 시 사라짐)
- 정말 귀엽다🥰 한동안은 오른쪽으로 해둘 거 같다

<br/>
<br/>
**4. 조회수 등록**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/242fcc4f-1bd3-4215-b89f-80096d480d6f)

- `/_includes/views/banner.html`
- [HITS](https://hits.seeyoufarm.com/)에서 제공해주는 기능이다.

<br/>
<br/>
**5. 사이드바 수정**

<div class="imageRow">
<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/839c6fac-3520-4e63-8580-83d2f9ad0654">
  <p>기존 사이드바</p>
  </div>
<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/04f5dfb2-48d4-4e9c-8adc-8e8a678dcd49">
</div>

- 기존 블로그 테마가 제공해주던 사이드바 디자인이 마음에 안들었다… 2 depth를 제공해주는 사이드바로 수정함
- 수정한 내용은 너무 길어서 따로 [게시글](https://lcqff.github.io/%EB%B8%94%EB%A1%9C%EA%B7%B8_%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8/2023/06/09/2depth_sidbar.html)에 포스팅했다.
