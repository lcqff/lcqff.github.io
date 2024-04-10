---
layout: post
title: 테스트 컨텍스트(Test Context) 줄이기
author: 팜
categories: Spring
tags: Spring 두레 성능 bug
sidebar:
---
## 개요

![디스커션](https://github.com/lcqff/lcqff.github.io/assets/71930280/1d6914dc-1319-4de6-b9ce-2e62c7f27fd0)

몇달 전 DooRe를 개발하던 중 위와 같은 Discussion이 올라왔다. 위 디스커션과 함께 [참고할 게시글](https://mangkyu.tistory.com/244)을 업로드해주셨는데 내가 백엔드를 공부한지 몇 달 안된터라 완벽한 원리를 이해하진 못했고, 대충 *~가 문제인데 ~를 통해 해결할수 있구나*라는 얄팍한 수준으로 이해했었다.😂

<br>

위 Discussion을 간단히 설명하자면 아래와 같이 코드를 수정하자는 말이다.

<br>

```java
@WebMvcTest(LoginController.class)
public class LoginApiDocsTest extends RestDocsTest {
    @MockBean
    protected LoginService loginService;
    
    ...
}
```

```java
@Import(RestDocsConfiguration.class)
@ExtendWith(RestDocumentationExtension.class)
public abstract class RestDocsTest extends ApiTestHelper {
    @MockBean
    protected LoginService loginService;

    ...그 외 모킹 대상들...
		
}
```

`LoginApiDocsTest와` 같은 RestDocs 테스트 클래스 모두가 상속하는 `RestDocsTest` 클래스 안에 RestDocs 테스트 클래스에서 필요한 Mocking 객체를 정의하는 것으로 **테스트 컨텍스트를 통일**시키고, 테스트 컨텍스트를 줄이는 것으로 테스트 시간을 줄이자는 내용이었다.

## 문제

우리는 위 디스커션을 받아들여 Mocking 객체를 모두 `RestDocsTest` 클래스에서 정의하는 식으로 테스트 코드를 작성해왔고, 그동안 별다른 문제점이 지적되지 않았다.

그렇게 몇달 후, 최근 어플리케이션 성능 향상에 대해 공부하며 테스트 컨텍스트 개수가 차이를 직접 비교해보고 싶었다. 마침 두레 프로젝트가 좋은 예제가 되어주었기에 테스트를 실행하여 확인해본 결과…. 

<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/a09b2b06-4f02-4b3d-938d-279eb8039200">
  <p>StudyApiDocsTest에서 생성된 컨텍스트</p>
</div>
<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/d9702938-6925-4168-af41-54b4865c267d">
  <p>ParticipantApiDocsTest에서 생성된 컨텍스트</p>
</div>

<br>

<img width="60%" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/744805d4-70c9-4d87-af53-6bafefec2dc7">

<br>

어라…? **각 RestDocs 테스트 클래스에서 테스트 컨텍스트가 생성**되고 있었다… 

<br>

직접 `logging.level.org.springframework.test.context.cache: debug` 설정을 통해 생성된 컨텍스트 개수를 살펴보자 아래와 같았다.  

```java
[DefaultContextCache@5e9ea380 size = 12, maxSize = 32, parentContextCount = 0, hitCount = 1911, missCount = 12, failureCount = 0]*
```

RestDocs 테스트를 제외한 테스트에서 테스트 컨텍스트가 4개 생성되고, RestDocs 테스트 클래스들이 총 8개니 <red>모든 RestDocs 테스트 클래스에서 테스트 컨텍스트가 생성</red>되고 있던 것이다🙂

## 애플리케이션 컨텍스트와 테스트 컨텍스트

문제 해결에 앞서 개념을 살펴보자. 

![애플리케이션 컨텍스트 상속 구조](https://github.com/lcqff/lcqff.github.io/assets/71930280/aa452006-e69e-4622-b3a8-232e7d427d57)

- [빈 팩토리(Bean Factory)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)
    - Spring Bean 컨테이너에 접근하기 위한 최상위 인터페이스
    - 스프링 빈 객체의 생성, 관리, 의존성 주입
- [**애플리케이션 컨텍스트(Application Context)**](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)
    - 애플리케이션 Configuration을 제공하는 핵심 인터페이스
    - Bean Factory를 상속받는다.
    - 스프링 빈 객체의 생성, 초기화, 의존성 주입, 라이프사이클 관리(Bean Factory 기능 상속)
    - XML, JavaConfig, 어노테이션(`@Configuration`) 등을 통해 애플리케이션의 환경 설정 정보를 제공, 관리
- [테스트 컨텍스트 프레임워크(Test Context Framwork)](https://docs.spring.io/spring-framework/reference/testing/testcontext-framework.html)
    - 테스트에 사용되는 애플리케이션 컨텍스트를 생성하고 관리하고 테스트에 적용해주는 기능을 가진 테스트 프레임워크(ex. JUnit, TestNG)
    - 실제 서버와 거의 동일한 구성으로 동작하는 애플리케이션 컨텍스트를 만들어준다.
- [**테스트 컨텍스트(Test Context)**](https://docs.spring.io/spring-framework/reference/testing/testcontext-framework.html)
    - 테스트를 실행할 때 필요한 애플리케이션 컨텍스트를 관리, 제공
    

애플리케이션 컨텍스트는 **스프링 애플리케이션의 전반적인 구성과 런타임 환경을 제어**하며, 테스트 컨텍스트는 한마디로 **테스트에 사용되는 애플리이션 컨텍스트를 관리해주는 객체**이다.

## 테스트 컨텍스트 캐싱

JUnit은 테스트 메소드를 실행할 때마다 새로운 객체의 테스트 클래스를 생성해 모든 테스트가 독립적으로 실행되도록 만든다. 

그러나 테스트가 독립적이기 위해서는 **매번 새로운 애플리케이션 컨텍스트가 필요**하며, 매번 새로운 빈을 생성하고 의존성을 주입하면 테스트 시간이 지연될 수밖에 없다.

<br>

**테스트 컨텍스트 캐싱**을 통해 위와 같은 문제를 해결할 수 있다. **동일한 테스트 구성을 가지는 테스트끼리 애플리케이션 컨텍스트를 공유**하면 새로운 애플리케이션 컨텍스트를 생성하지 않게된다.

![애플리케이션 컨텍스트 캐싱](https://github.com/lcqff/lcqff.github.io/assets/71930280/ae4531de-fc45-4603-929a-c46dfacb2f8b)

위 이미지에선 N개의 테스트가 한 개의 애플리케이션 컨텍스트를 공유하며, 총 2개의 컨텍스트가 생성되게 된다.

## 원인

제일 처음에 제시된 디스커션에서 말하는 내용 또한 각 테스트 클래스의 구성을 통일시켜, **테스트 컨텍스트 캐싱**을 통해 새로운 애플리케이션 컨텍스트를 생성하지 않도록 만들자는 의미였다.

스프링은 동일 구성이라면 컨텍스트를 재사용하되, 애플리케이션 컨텍스트 내부에 변경이 발생한다면 새로운 애플리케이션 컨텍스트를 생성하도록 한다.

{: .box-yello}
- `@MockBean` `@SpyBean`
- `@TestPropertySource`
- `@ConditionalOnX`
- `@WebMvcTest(~.class)`
- `@Import`
- 등등…

이때 `@WebMvcTest(~.class)`는 해당 컨트롤러와 관련된 빈들만을 로드하도록 컨텍스트를 제한시키고, `@MockBean` `@SpyBean`는 특정 빈을 Mock이 적용된 빈으로 등록하는 것으로 컨텍스트 구성을 변경시킨다.

<br>

```java
@WebMvcTest(LoginController.class)
public class LoginApiDocsTest extends RestDocsTest {
    @MockBean
    protected LoginService loginService;
    
    ...
}
```

```java
@Import(RestDocsConfiguration.class)
@ExtendWith(RestDocumentationExtension.class)
public abstract class RestDocsTest extends ApiTestHelper {
    @MockBean
    protected LoginService loginService;

    ...그 외 모킹 대상들...
		
}
```

문제 원인은 간단했다. `@MockBean`과 `@WebMvcTest` 둘 다 새로운 애플리케이션 컨텍스트를 생성하는데 우리는 **RestDocsTest에 필요한 모든 모킹 객체를 정의하기만 하고 `@WebMvcTest`를 추가하지 않은 것**이다.

<br>

따라서 수정된 코드는 아래와 같다.

```java
public class LoginApiDocsTest extends RestDocsTest {
  ...
}
```

```java
@Import(RestDocsConfiguration.class)
@ExtendWith(RestDocumentationExtension.class)
@WebMvcTest({
    LoginController.class,
    ...그 외 빈들을 로드할 컨트롤러...
})
public abstract class RestDocsTest extends ApiTestHelper {
    @MockBean
    protected LoginService loginService;

		...그 외 모킹 대상들...
}
```

## 결과

```java
*수정 전 컨텍스트
[DefaultContextCache@5e9ea380 size = 12, maxSize = 32, parentContextCount = 0, hitCount = 1911, missCount = 12, failureCount = 0]

수정 후 컨텍스트
[DefaultContextCache@a1b7549 size = 5, maxSize = 32, parentContextCount = 0, hitCount = 1925, missCount = 5, failureCount = 0]*
```

RestDocs 테스트 클래스가 한개의 애플리케이션 컨텍스트만을 공유하는 것으로 RestDocs 테스트 외에서 생성되는 애플리케이션 컨텍스트 4개 + RestDocs 테스트 클래스들이 공유하는 애플리케이션 컨텍스트 1개 = <red>총 5개</red>의 컨텍스트로 줄어들었다. 7개의 애플리케이션 컨텍스트가 줄어들은셈….👍 

또한 테스트 실행시간 또한 <red>11sec 415ms에서 8sec 617ms</red>로 상당히 감소했다😄 만족

---

> 배운 점: 어떤 성능문제를 해결한 후에는 성능 차이를 비교, 확인하자…😂


<br><br><br>

참고: 
- https://mangkyu.tistory.com/151
- https://mangkyu.tistory.com/202
- https://mangkyu.tistory.com/244