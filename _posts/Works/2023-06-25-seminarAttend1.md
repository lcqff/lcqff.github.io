---
layout: post
title: "출석체크 기능 제작 1- 기획"
author: 파녀미
categories: Works
banner:
  image: https://github.com/lcqff/lcqff.github.io/assets/71930280/537723bc-fa2b-4642-9747-d9574c43316d
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: keeper
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
  .slash {
  background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg"><line x1="0" y1="100%" x2="100%" y2="0" stroke="lightGray" /></svg>');
}
</style>

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/537723bc-fa2b-4642-9747-d9574c43316d)

## 서론

Keeper라는 부산대학교 보안동아리에서 난데없이 개발을 하고 있는 나는 이번 홈페이지 리뉴얼 작업에서 출석체크 페이지 개발을 맡았다.<br/>

사실 버전 1에서도 동일한 기능을 개발했던 터라 개발이 쉬울 줄 알았는데... 여러가지 바뀐 점 + TypeScript와 eslint 사용으로 꽤 많은 부분을 수정해야 했다. (버전 1 코드가 개발새발이어서 그런 것도 있다.)

앞으로 해당 기능을 어떻게 개발했는지, 어떤 오류가 있었는지, 어떤 부분이 수정되었는지는 실시간으로 게시글로 작성할 생각이다. <br/>

이 게시글을 포함 앞으로 올라올 출석페이지 관련 게시물은 앞으로 해당 기능을 유지, 보수 할 회원의 편의를 목적으로 한다.

## FlowChart

![회장 flowchart](https://github.com/lcqff/lcqff.github.io/assets/71930280/dfd94d49-7b43-40d8-a5ef-7a2e4dce7cb5)
![회원 flowchart](https://github.com/lcqff/lcqff.github.io/assets/71930280/0b619336-fec6-4e3a-ba5c-387107e607da)

## Permission

<table style="text-align: center; display:center;">
    <thead>
        <tr>
            <th style="text-align: center;">대구분</th>
            <th style="text-align: center;">중구분</th>
            <th style="text-align: center;">사용자</th>
            <th style="text-align: center;">보기</th>
            <th style="text-align: center;">수정</th>
            <th style="text-align: center;">삭제</th>
            <th style="text-align: center;">쓰기</th>
            <th style="text-align: center;">비고</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=16>출석체크 페이지</td>
            <td rowspan=4>출석 시간 설정</td>
            <td>회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td >O</td>
            <td rowspan=4></td>
        </tr>      
        <tr>
            <td>부회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>  
        <tr>
            <td>서기</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>
        <tr>
            <td>회원</td>
            <td>X</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>X</td>
        </tr>
        <tr>
            <td rowspan=4>출석 시작</td>
            <td>회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
            <td rowspan=4></td>
        </tr>      
        <tr>
            <td>부회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>  
        <tr>
            <td>서기</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>
        <tr>
            <td>회원</td>
            <td>X</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>X</td>
        </tr>
        <tr>
            <td rowspan=4>출석</td>
            <td>회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
            <td rowspan=4 style="text-align: left;">회원은<br/> 출석 기능만 접근 가능</td>
        </tr>      
        <tr>
            <td>부회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>  
        <tr>
            <td>서기</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>
        <tr>
            <td>회원</td>
            <td>X</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>X</td>
        </tr>
        <tr>
            <td rowspan=4>관리 페이지 이동</td>
            <td>회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
            <td rowspan=4></td>
        </tr>      
        <tr>
            <td>부회장</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>  
        <tr>
            <td>서기</td>
            <td>O</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>O</td>
        </tr>
        <tr>
            <td>회원</td>
            <td>X</td>
            <td class="slash"></td>
            <td class="slash"></td>
            <td>X</td>
        </tr>
    </tbody>
</table>

## UI

<p style="color:red; font-weight: bold;">UI 부분은 개발을 진행하며 추가할 사항이 존재하며, 실시간으로 업데이트 될 예정이다.</p>
<br/>
<br/>

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/d67a39eb-f98d-4172-96a5-8a332ced111a)
<br/>
**Page Title**: 세미나 출석<br/>
**Author**: 파녀미<br/>
**Screen Path**: HOME> 참여 마당> 세미나 출석<br/>
**권한**: 회장, 부회장, 서기<br/>

<table>
  <thead >
    <th colspan=2 style="text-align: center;">Description</th>
    <th style="text-align: center;">Check Point</th>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>세미나 관리 페이지로 이동</td>
      <td rowspan=4>좌측에 예정된 세미나 카드가, 우측에 종료된 세미나 카드가 배치된다.<br/><br/>
새로고침/로그아웃 시에도 타이머가 리셋되지 않도록 한다.<br/><br/>
출석을 시작한 회원은 자동으로 출석처리하도록 한다.<br/><br/>
출석 시간 옵션은 5분, 10분, 15분, 20분이 있다</td>

  </tr>
  <tr>
    <td>2</td>
    <td>출석 마감 시간 설정</td>
  </tr>
  <tr>
    <td>3</td>
    <td>지각 마감 시간 설정</td>
  </tr>
  <tr>
    <td>4</td>
    <td>출석 시작</td>
  </tr>

  </tbody>
</table>

<br/>

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/451f8040-761a-4946-9cf4-325ff0a29922)
<br/>
**Page Title**: 세미나 출석<br/>
**Author**: 파녀미<br/>
**Screen Path**: HOME> 참여 마당> 세미나 출석<br/>
**권한**: 회장, 부회장, 서기, 회원<br/>

<table>
  <thead >
    <th colspan=2 style="text-align: center;">Description</th>
    <th style="text-align: center;">Check Point</th>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>세미나 관리 페이지로 이동</td>
      <td rowspan=4>지각시간 마감 전까지 출석하지 않을 경우 결석처리 한다.<br/><br/>
<p style="color:red;">출석코드 제출 횟수는 5회로 제한한다.</p><br/>
출석코드 제출 횟수가 5화를 초과하면 모달을 띄우고 결석 처리한다.<br/><br/>
생성되었으나 시작되지 않은 세미나의 시간은 --:--으로 보인다.<br/><br/>
회장,부회장 혹은 서기 권한을 가지지 않은 회원은 세미나 관리 버튼을 볼 수 없다.
</td>

  </tr>
  <tr>
    <td>2</td>
    <td>출석코드 입력</td>
  </tr>
  <tr>
    <td>3</td>
    <td>출석 버튼</td>
  </tr>
    <tr>
    <td>4</td>
    <td>모달 확인 버튼</td>
  </tr>
  </tbody>
</table>
