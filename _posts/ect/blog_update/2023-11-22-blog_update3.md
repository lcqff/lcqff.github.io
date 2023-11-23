---
layout: post
title: "23.11.22 블로그 업데이트3"
author: 퍄녀미
categories: 블로그_업데이트
sidebar: []
---

블로그를 오랜만에 쓰려니 너무 불편하고 거슬리는 부분이 많다😂 블로그 다시 시작하자마자 하는 짓이 테마 수정이라니...
그러나 깃허브 블로그는 이렇게 테마를 뜯어고치며 블.꾸 하는 게 묘미인 거 같다~

## 추가 기능

<br/>
<br/>
**1. 댓글 플랫폼 변경**

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/45457685-23e0-4f3d-b033-aeaa97fd3cbd)

- 기존에 사용하던 댓글 플랫폼을 Disqus에서 Utterances 변경했다. 사유!!!!

  1. 너무 못생김........

     ![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/d399cd8d-8e53-4981-9a8c-f84e0d4c682d)
  
     으윽! 윽!
     숨막히게 못생겼다. 사실 이 이유만으로도 댓글 플랫폼을 바꾸려고 생각하고 있었다 (귀찮아서 실행하지 않았을 뿐) 그에 비해 Utterances는 정말 아름답다.

  2. 결정적으로 댓글이 안달린다😂<br/>
     
     사실 테스트용으로 만든거지 누군가가 댓글을 달거라 생각하고 만든 기능이 아니여서 전혀 신경쓰고 있지 않았는데...
     얼마전 친구와 카공을 하다가 댓글 기능이 아예 작동하지 않는다는 걸 알게 되었다😂
     <br/>사라진 친구의 댓글에게는 애도하는 마음 뿐이다... 어떻게 살려보려 했으나 Disqus을 사용한 블로그 게시글들이 다 오래된 게시글 뿐이고 그때에 비해 Disqus 사이트가 꽤 많이 바뀌어서 결국 해결 방법은 알아낼 수 없었다.
     아무튼 Disqus구리다 쓰지마라

- Issue로 댓글 관리가 가능하다💕
  ![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/47df105b-8227-4d7e-bbf1-11be7e35700e)
  - Disqus로 댓글을 유실하고 나니 정말 천사같은 기능이다. 위와 같이 Github레파지토리 이슈로 댓글관리가 가능하다.

<br/>
<br/>
**2. 박스 css 변경**
<div class="imageRow">
  <img alt= "이전 박스 css" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/2ba6e79b-e6cf-4bfe-8e05-0d41b84794b3">
  <img size="20%" alt="새로운 박스 css" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/6a787c84-4456-45c8-8871-614ff6ae7aee">
</div>
- 기존의 박스 css가 너무 안예뻐서 안쓰게 되길래 이번에 조금 가독성있고 예뻐보이도록 css를 바꿨다.
- 좀 둥글게 깎고 패딩을 더 넓히고 색을 연하게 만들었다.. 이제 좀 쓸만해진 거 같다.

<br/>
<br/>
**3. 포스트 내 이미지 캡션/이미지 행 정렬**
- 사실 기존에도 html로 이미지 캡션과 중앙 정렬을 사용하고 있긴 했다. 그러나 매번 css를 사용할때마다 해당 포스트에 동일한 스타일 코드를 복붙해야하는 현상이... 귀찮아서 해결 안하고 있었다.
- 이번에 레이아웃 설정 파일에 아예 이미지 캡션과 이미지를 행으로 정렬하는 기능을 작성해놓았다.
- `layout.scss`에서 `.post-content`에 스타일 클래스를 추가했다.
- **{: .imageRow}**
- **{: .captionedImg}**


<br/>
<br/>
**4. 글씨 컬러 지정**
- <span><red>빨간색 글씨</red>와 <blue>파란색 글씨</blue>를 추가했다.</span>
- 이것도 그냥 html로 작성하고 있었는데 이번에 그냥 레이아웃 설정에 넣어버렸다.
- &#60;red&#62;로 감싸면 해당 글자의 색이 바뀌도록 했다~ 
- `layout.scss`에서 `.post-content`에 태그를 추가했다.


<br/>
<br/>
**5. 글씨 마진 변경**
<div>
  <img alt="변경 전" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/4fbc9455-4721-41e6-b538-9c3cf4971473">
  <img alt="변경 후" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/43c0b1a1-6de0-410a-81cf-93b4e1c00c8c">
</div>
{: .imageRow}
<div>
<img alt="변경 전" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/c5ff3666-0c23-445b-9bf5-0580ea8a6c89">
<img alt="변경 후" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/0c070747-dd6a-43cf-a3a0-f0d6012e3979">
</div>
{: .imageRow}

- 기존 블로그에선 줄바꿈시 여백을 넣지 않았음에도 동일 문단의 글이 공백으로 띄어놓은 것처럼 갈라지는 문제가 있었다.
- 이것도 귀찮아서 안바꾸고 있었는데 글 작성이 너무 불편해서... 이번에 고쳤다. 강제적으로 줄 여백이 생기는 것보다 내가 직접 여백을 삽입하며 글을 적는 것이 글.꾸 하는 데 훨씬 편한거 같다😂
- 하는 김에 리스트의 마진이 더 넓은게 가독성에 좋을 거 같아 더 넓혔다.
- `layout.scss`에서 `.post-content`의 p와 li 마진을 변경했다.