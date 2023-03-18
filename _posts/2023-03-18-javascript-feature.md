---
layout: post
title: "Javascript의 특징"
author: 사탄
categories: Javascripy
banner:
  image: https://user-images.githubusercontent.com/71930280/226112111-c23343a0-ceee-44ec-8f1e-b3802d49460d.png
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: javascript
sidebar: []
---

javascript는 웹 기반 응용 프로그램 및 웹 브라우저를 만들기 위해 사용되는 언어입니다. javascript를 사용하면 동적 및 대화형 웹 콘텐츠를 만들 수 있습니다.
<br/><br/>

## Javascript의 특징

#### 싱글 스레드 언어

javascript는 싱글 스레드 언어로 하나의 호출 스택과 하나의 메모리 힙을 사용합니다.<br/>
따라서 javascript는 한 테스트가 완료되기 전까지 다른 테스크를 수행하지 않습니다.

#### 비동기적 동작 가능

그러나 비동기적으로 동작한다는 점 또한 javascript의 특징이라 말할 수 있습니다.

> **비동기적 실행이란?** <br/
  코드들이 비동기적으로 실행된다는 것은 코드가 실행되는 순서와 결과가 발생하는 순서가 일치하지 않는 것.

이는 javascript 코드가 브라우저를 통해 실행되기에 가능한 것입니다.<br/>
javascript의 함수들을 싱글 스레드 형식으로 처리하는 <span style="background-color:#fff5b1;">**콜스택**</span>과 별개로 브라우저에 비동기적으로 실행된 콜백 함수들을 보관하는<span style="background-color:#fff5b1;">**콜백큐**</span>가 존재합니다.<br/>
<span style="background-color:#fff5b1;">**이벤트 루프**</span>는 콜스택의 상태를 관찰하다가 **콜스택**이 비었을 경우 **콜백큐**에서 대기중인 함수를 **콜스택**으로 불러오게 됩니다.<br/>
따라서 먼저 실행된 비동기 함수가 **콜백큐**에서 대기하는 동안 그 이후에 실행된 함수가 **콜스택**에서 처리되어 먼저 끝나게 되는 것입니다.

#### 인터프리터 언어

javascript는 인터프리터 언어라는 특성을 가지고 있습니다. 인터프리터 언어는 소스 코드를 한줄씩 읽어들여 바로 실행합니다. 이는 javascript의 코드가 동적으로 실행됨을 말하며, 변수의 타입이 런타임에서 결정되게 합니다.
<br/><br/>

## Javascript의 편리성

javascript는 위와 같은 특징으로 인한 여러 편리성을 가지고 있습니다. Javascript는 싱글스레드로 동작하기 때문에 자원 접근에 대한 동기화를 신경쓰지 않아도 되며, 비동기적 처리를 지원하기 때문에 프로그램의 성능을 높일 수 있습니다.

또한 변수의 타입이 런타임시 결정되기 때문에 변수의 타입 지정에 대해 자율성이 높습니다. 프론트 엔드, 백엔드, 어플리케이션 등 다양한 용도로 사용될 수 있다는 점 또한 Javascript의 장점입니다.
<br/><br/>

## Javascript의 불편성

하지만 Javascript는 이러한 특징으로 인한 단점또한 가지고 있습니다.<br/>
Javascript의 동적인 특징은 하나의 변수에 여러 타입을 지정할수 있도록 하며, 이는 변수에 예상치 못한 타입을 지정시켜 오류를 발생시킬 수 있습니다. 이러한 오류는 런타임시에만 발견 가능하기에 에러를 찾기 어렵습니다.<br/>
또한 javascript는 인터프리터 언어이기 때문에 컴파일러 언어에 비해 성능이 느릴 수 있습니다.
