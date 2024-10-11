---
layout: post
title: "스프링 프로젝트 생성과 실행"
author: 파녀미
categories: Backend-Study
tags: Spring
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

{: .box-yello}
읽기 전 [이 게시물](https://lcqff.github.io/cs/2023/06/07/MVC.html)을 먼저 읽고 오는 것을 추천한다.

## 스프링 프로젝트 생성

[start.spring.io](https://start.spring.io/)

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/ae29bd8f-e717-4b0e-8270-35c52a6d3561)
Dependancies로 Spring Web과 Thymeleaf를 추가한 뒤 build해준다.

- Spring Web: 웹프로젝트용 스프링을 개발하는데 필요한 모듈
- Thymeleaf: HTML 만들어주는 템플릿 엔진

<br/>
<div class="imageRow">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/6189ac49-3ae8-4fc5-9ec2-3926ebe2b2b6">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/2ca30bec-d74b-43d9-abed-59a0488f3c94">
</div>

## 스프링 실행

### 정적 페이지

```java
package y2h.Springbegin;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBeginApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringBeginApplication.class, args);
	}

}
```

Spring을 실행시켜준다.

실행버튼을 누르면 터미널의 Spring 화면과 함께 `localhost:8080`에 에러 페이지가 뜨는 것을 확인할 수 있다. 웰컴 페이지가 없어서 그러는 것이니 만들어주자.

Spring Boot에서는 `static/index.html` 을 올려두면 [Welcome page](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#web.servlet.spring-mvc.welcome-page) 기능을 제공하도록 되어있다.

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/41b1c2d9-9a05-4f6f-b55b-2f7ff4296ea5)

```java
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/45147c46-dd97-469f-a332-9fb239255216)

위 코드는 **static한**(정적인) 웰컴 페이지를 제공해준다. 동적인 페이지를 만들기 위해서는 템플릿 엔진을 사용해야한다.

### Thymeleaf 템플릿 엔진

```java
package y2h.Springbegin.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {
    @GetMapping("hello") //웹페이지에서 /hello로 이동할시 메서드를 호출
    public String hello(Model model) {//요청 처리 로직
        model.addAttribute("data","hello!!");
        return "hello";
    }
}
```

<p style="color:gray">HelloController.java</p>
`@GetMapping`은 HTTP GET 요청을 처리하는 핸들러 메서드를 정의하는 데 사용되는 어노테이션이다.<br/>
핸들러 메서드 `hello`에 `@GetMapping` 어노테이션을 적용한다. 이때 어노테이션의 매개변수로는 해당 요청을 처리할 URL인 `/hello`를 지정했다.

클라이언트로부터 /hello경로의 GET 요청이 올시, 요청은 `hello()` 메서드에 의해해 처리된다.

`model.addAttribute("data","hello!!");`은 Spring MVC에서 사용되는 메서드로, 컨트롤러에서 모델에 데이터를 추가하는 역할을 한다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
    <p th:text="'안녕하세요. ' + ${data}">안녕하세요. 손님</p>
  </body>
</html>
```

<p style="color:gray">resources/templates/hello.html</p>
th: thymeleaf 문법

위는 data 속성 값을 **동적**으로 출력하는 HTML 코드이다.
모델에 data라는 속성이 존재할시 해당 속성값을 모델에서 가져와 출력한다.
`HelloController.java`에서 우리는 data의 값으로 "hello!!"를 입력했다. 따라서 결과적으로 출력되는 값은 **안녕하세요. hello!!**가 된다.
![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/8b2e7bd9-cd18-44e1-8c1b-f26d5fc7ebd4)

### 실행

터미널 입력

{: .box-yello}
`./gradlew build`<br/>
`cd build/libs`<br/><br/>
![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/d87531f3-fae2-4198-ab33-74e081635814)
`java -jar Spring-begin-0.0.1-SNAPSHOT.jar`

---

## 원리

### 정적 컨텐츠

<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/981b8a6a-a335-46ea-bf3c-77be439b5d0b">
  <a href="https://www.inflearn.com/course/lecture?courseSlug=%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8">https://www.inflearn.com/course/lecture?courseSlug=스프링입문</a>
</div>

웹브라우저가 /hello-static.html으로 이동하며 `hello-static.html` 파일을 요청한다.<br/>
요청을 받은 내장 톰켓 서버는 스프링 컨테이너에서 `hello-static`과 매칭되는 컨트롤러를 찾는다.<br/>
만일 매칭되는 컨트롤러가 존재하지 않을 경우 **resources**에서 `hello-static.html` 파일을 찾아 반환한다.

정적 컨텐츠는 static 폴더 내부에 html파일을 생성하는 것으로 생성할 수 있다.

### MVC 템플릿 엔진

<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/93fd4783-fbe6-4f15-b826-aef093668f4e">
  <a href="https://www.inflearn.com/course/lecture?courseSlug=%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8">https://www.inflearn.com/course/lecture?courseSlug=스프링입문</a>
</div>

웹브라우저에서 `/hello`로 이동하면 내장 톰켓 서버는 `GetMapping`을 사용해 스프링 컨테이너에서 `/hello`경로와 동일한 메서드를 찾아 매칭하고, 해당 메서드를 실행시킨다.

실행된 메소드가 리턴 값으로 문자열을 반환하면 viewResolver가 viewName을 매핑해 **templates** 폴더 아래에 매칭된 html을 랜더링한다. (`resorces: templates/` + {ViewName} + `.html`)

- 웹 브라우저: /hello로 url 이동
- 내장 톰켓 서버: GetMapping으로 경로와 동일한 메서드 찾아 실행
- 스프링 컨테이너
  - 메서드: 로직을 실행시킨다. 파라미터로 넘어온 data를 model에 넘기고 문자열을 return한다.
  - viewResolver: model의 키 값을 이용해 model로부터 값을 가져온다, 또한 반환된 문자값을 기반으로 viewName을 매핑해 templates 폴더 아래에 있는 html을 화면에 표시한다.

### API

```java
@GetMapping("hello-string") //api방식
@ResponseBody //응답 body 부분에 반환값을 직접 넣어준다.
public String helloString(@RequestParam("name") String name) {
    return "hello " + name;
}
```

<div class="imageRow">
  <div class="captionedImg">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/8b2e7bd9-cd18-44e1-8c1b-f26d5fc7ebd4">
    <p>mvc 방식 코드</p>
  </div>
  <div class="captionedImg">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/2d63371d-f2d8-4458-b214-30693515088d">
    <p>@ResponseBody를 사용한 코드</p>
  </div>
</div>

`@ResponseBody`를 사용해 반환값을 HTTP 응답의 body 부분에 직접 넣었다.
또한 `@RequestParam`을 사용해 클라이언트가 URL의 쿼리부분에 name이라는 이름의 요청 파라미터값을 전달하도록 했다.

```java
@GetMapping("hello-api")
@ResponseBody
public  Hello helloApi(@RequestParam("name") String name ){
    Hello hello = new Hello();
    hello.setName(name);
    return hello;
}
static class Hello {
    private String name;

    public void setName(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}
```

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/8e6eae35-1db1-4615-b2af-1efab9f7853d)

위 코드와 다르게 문자열이 아닌 **객체**를 반환하는 코드다. 그 결과값으로 HTTP의 body에 Json값이 삽입되었다.

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/0da19ed8-efda-4257-9ba7-a42fcb698bcd)

`@ResponseBody`가 사용된 경우 viewResolver 대신 **HttpMessageConverter**가 작동한다.
이때 메서드가 반환한 값이 문자열인가 객체인가에 따라 다른 컨버터가 실행된다.

- 단순 문자 반환: StringConvertor 동작
- 객체 반환: JsonConverter를 통해 json으로 변환됨.
