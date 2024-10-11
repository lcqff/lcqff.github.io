---
layout: post
title: 구글 소셜 로그인 에러 해결(Cors, Redirect uri)
author: pam
categories: DooRe
description: "[troubleShooting] Doore에서 발생한 구글 소셜 로그인 에러 troubleShooting"
tags: Spring Backend DooRe troubleShooting
image: https://github.com/lcqff/lcqff.github.io/assets/71930280/90e41dda-7060-491a-9c6b-4f7200b1d561
toc: true
---

## 개요

![두레 메인](https://github.com/lcqff/lcqff.github.io/assets/71930280/7b4f62c4-47e8-4332-8289-dbda3c33e0a8)

두레의 소셜 로그인에서 <red>CORS 에러</red>가 발생한다는 프론트의 보고가 들어왔다. 

<br>

구글 소셜 로그인은 현재 팀을 나간 팀원이 몇 달 전에 작업한 내용이었다. 

해당 기능이 개발되던 당시에는 클라이언트 UI 작업이 되지 않아 제대로된 테스트가 이루어지지 못했고, 직접 클라이언트를 통해 로그인을 시도하자 오류가 발생한 것이다. (인터넷 URL 창에 직접 uri를 입력하여 정상 작동하는지 테스트했다 들음)

백엔드는 이미 권한 처리가 완료돼있었기에 로그인을 하지 못하면 프론트에서 작업하기가 어려운 상황이었다.

해당 포스팅에는 구글 소셜 로그인에서 발생한 **Cors, redirection url**의 두가지 문제를 해결한 내용을 기록해두었다. ~~(그 사이의 많은 삽질은 생략했다…)~~

## Cors 오류


![cors](https://github.com/lcqff/lcqff.github.io/assets/71930280/bd1ed749-452b-4d21-b3ed-ce463c1d93ab)

[가짜 Cors 해결(false cors)](https://lcqff.github.io/keeper/2024/03/30/false-cors.html)

(Cors 오류에 대한 설명은 위 포스팅을 참고하자.)

### application.yml 수정

```yaml
cors:
  allow:
    origins: http://localhost:3000
    methods: GET, POST, PUT, DELETE, PATCH, OPTION
```

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Value(value = "${cors.allow.origins}")
    private String[] allowedOrigins;

    @Value(value = "${cors.allow.methods}")
    private String[] allowedMethods;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins(allowedOrigins)
                .allowedMethods(allowedMethods)
                .allowedHeaders("Origin", "Content-Type", "Accept", "Authorization")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

최우선적으로 Cors 설정 관련 문제를 들여다보자. 

**allowedOrigins**에 프론트 로컬 경로만 설정돼있고 실제 개발 서버가 연결돼있지 않은 것을 확인할 수 있었다.

<br>

```yaml
cors:
  allow:
    origins: http://localhost:3000, https://www.doore.kro.kr
    methods: GET, POST, PUT, DELETE, PATCH, OPTION
```

개발서버를 추가해주자. 



### 서브모듈 수정

![여전](https://github.com/lcqff/lcqff.github.io/assets/71930280/90e41dda-7060-491a-9c6b-4f7200b1d561)

그러나 application.yml의 allowOrigin을 수정해도 여전히 동일한 에러가 발생했다.

이쯤해서 Doore의 민감정보 처리 방법에 대해 간단하게 알아보자...

#### Submodules

간과했던 사항은 <red>로컬과 개발서버, 운영서버는 서로 다른 설정 파일을 읽는다</red>는 점이다.

<br>

 환경이 나누어져 있는 이상 데이터베이스(또는 스키마)와 같이 **각 환경별로 다르게 사용해야 할 설정**이 있기 마련이며, 개발 서버와 운영서버에는 api-key또는 데이터베이스 비밀번호와 같이 **외부인에게는 공개해선 안되는 민감 정보**가 존재한다. 

당연하게도 이러한 설정을 통일하기 보다는 각 환경별로 분리하는 쪽이 운영에 훨씬 도움이 된다.

따라서 두레에서는 환경에 따른 각 설정 정보를 다음과 같이 분리했다.

- `application.yml` → 로컬 설정
- `application-dev.yml` → 개발 환경 설정
- `application-prod.yml` → 운영 환경 설정
{: .box-yello}

<cap>(설정파일 네이밍에 관해서는 아래 Profile 파트에서 설명하겠다.)</cap><br>

현재 두레 레포지토리는 public 상태로, `application.yml`은  외부에 공개돼있다. 
그러나 개발 환경 설정과 운영 환경 설정은 민감한 정보를 담은 파일로 <blue>Private</blue>하게 관리해야 한다.

따라서 Doore에서는 **Submodule**을 사용하여 민감 정보를 **private 레파지토리에 보관**하고 있다. 


#### Spring Profile

그렇다면 어떻게 운영 환경에서는 `application-prod.yml`의 설정을 가져다 사용하고, 개발 환경에서는 `application-dev.yml`의 설정을 가져다 쓸 수 있도록 할 수 있단 말인가?

이것은 <red>Profile</red>에 달려있다.

<div class="callout">
  <div>💡</div>
  <div>
	<strong>Profile</strong><br/>
    특정 환경에 맞게 애플리케이션 설정을 다르게 적용할 수 있도록 해주는 기능
  </div>
</div>


![profiles](https://github.com/lcqff/lcqff.github.io/assets/71930280/61a6a10e-92cc-4231-acef-bc2674b296e1)
<cap>https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/boot-features-profiles.html#boot-features-profiles </cap><br>

스프링 부트에서는 `spring.profiles.active` 설정을 통해 활성화 할 프로파일을 설정할 수 있다.

이 설정은 하나의 yml 파일에서 로컬,개발,운영 환경별 설정을 나눌 때 사용할 수도 있고, 어플리케이션 실행시 Java 시스템 프로퍼티를 통해 설정할 수도 있다.

<br>

![dockerfile](https://github.com/lcqff/lcqff.github.io/assets/71930280/1c36bcf2-adc1-4dcb-9632-edfc9dbb3c72)

두레에서는 Docker를 사용하여 애플리케이션을 실행하고 있다.

DockerFile을 확인해보면 어플리케이션 실행시 **`-Dspring.profiles.active` 시스템 프로퍼티를 사용하여 프로파일을 dev로 설정하고 있음**을 확인할 수 있다.

<br>

![propertiees](https://github.com/lcqff/lcqff.github.io/assets/71930280/c455154f-dc1d-481d-9294-4fe7bab36674)
<cap>
https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/boot-features-external-config.html#boot-features-external-config-profile-specific-properties</cap><br>

SpringBoot 설정파일 네이밍 규칙에 의하여 어떤 프로파일의 설정 파일의 이름은 `application-{profile}.properties`가 된다.

따라서 profile이 dev로 설정된 개발서버에서 사용하는 설정파일은 `application-dev.yml`이 될 것이다.


### 해결

![cors 해결](https://github.com/lcqff/lcqff.github.io/assets/71930280/30dc8a3e-a85a-4b93-b8f3-dcba6a4ca704)

확인해보니 서브모듈의 Allow origin은 여전히 로컬서버를 향하고 있는 상태였다. 

따라서 배포 서버 주소를 추가해주는 것으로 Cors 에러를 해결할 수 있었다.

[서브모듈 수정 방법](https://lcqff.github.io/doore/2024/07/13/submodule.html)

## redirect_uri_mismatch 오류


![redirect uri mismatch](https://github.com/lcqff/lcqff.github.io/assets/71930280/8cbabbf8-ed87-4b09-b463-69b1803186ba)

Cors 에러를 수정하자 이번엔 500에러가 발생했다...

검출되지 못한 500에러가 지금 나를 덮친다……

<br>

![ec2 log](https://github.com/lcqff/lcqff.github.io/assets/71930280/20e732c9-62dd-45e7-8b75-f721d82ce02b)

EC2에 접속해 서버 로그를 확인해보자. **redirect_uri_mismatch** 오류가 뜨는 걸 확인할 수 있었다.


코드를 살펴보며 두레 구글 소셜 로그인 코드에서 `redirect_uri`를 어디에 사용하고 있는지 확인했다.

<br>

1. 클라이언트에서 구글로 승인(인가코드 발급) 요청을 보낼 때
    1. 사용하고 있는 rediect_uri: `https:/www.doore.kro.kr/oauth2/code/google`
2. 백엔드에서 구글로 토큰 요청을 보낼 때
    1. 사용하고 있는 redirect_uri: `https://www.doore.kro.kr`
{: .box-yello}

위의 두 군데에서 redirect-uri가 사용되고 있음을 확인했다.

<br><br>

![dev doc](https://github.com/lcqff/lcqff.github.io/assets/71930280/37b3b958-e53a-4f10-87c7-a5f5bf2f78c7)
<cap>https://developers.google.com/identity/protocols/oauth2/web-server?hl=ko#httprest_1</cap><br>


구글 oauth2 문서에서 설명하길, **[credentials page](https://console.cloud.google.com/apis/credentials/oauthclient/470276714816-e3q16jtp6a5tmogt0tt1l1d0u8brhr65.apps.googleusercontent.com?authuser=2&project=doore-411801)의 승인된 리디렉션 URI와 일치하지 않은 redirection uri가 사용된 경우** redirect_uri_mismatch 오류가 발생할 수 있다고 한다. 

### Redirect uri

credentials page의 승인된 리디렉션 URI를 확인해보았다.


![승인된 redirection](https://github.com/lcqff/lcqff.github.io/assets/71930280/9dfb96a0-3680-4872-b1f8-4aaf201c9905)

이미 여러가지 리디렉션 URI가 등록되어 있었다.

그러나 `https:/www.doore.kro.kr/oauth2/code/google`를 제외한 `https://www.doore.kro.kr`는 등록되어있지 않았음을 확인하고 새로 등록해주었다. (이후에 안쓰는 리디렉션 URI들은 정리하는 걸로...)

google credentials page에 새로 등록된 리디렉션 URI는 실제로 적용되기까지 5분에서 n시간이 소요된다. 당연하게도 실제 적용됐다고 알림도 안온다. 그냥 해봤는데 안되면 아직 적용이 안됐거니 하고 기다려야하는 잔인한 시스템… (여기서 굉장히 많은 시간을 낭비했다)

아무튼 결과적으로 리디렉션 URI에 값을 추가하는 걸로는 **문제가 해결되지 않았다.**

### 해결

 구글 개발 문서 상에는 등록되지 않는 리디렉션 URI를 사용하여 발생하는 문제라 되어있긴 하나, 이쯤되니 **리디렉션 URI 등록 문제가 아니어보였다.**

다행히도(다행이라기엔 너무 많은 삽질을 거쳤지만) 이전에 개인 공부로 카카오 소셜 로그인을 구현해본적이 있다. 

구글 소셜 로그인을 개발하신 팀원분은 URL창에 get 요청을 전송하는 식으로 테스트를 하셨지만 나는 클라이언트로 직접 소셜 로그인이 성공하는 것까지 확인했기 때문에 **두레의 구글 소셜 로그인 코드와 내가 작성한 카카오 소셜 로그인 코드를 비교해가며 문제를 찾는 것이 좋겠다 판단**했다. (플랫폼이 다르긴 해도 소셜 로그인 원리는 거의 다르지 않을 것이기 때문에)


![카카오 소셜 로그인](https://github.com/lcqff/lcqff.github.io/assets/71930280/2a7f1d1a-9360-414f-9e8d-ec18aa3f742e)
<cap>https://lcqff.github.io/dotted/2024/05/01/kakao-login.html </cap><br>

내가 카카오 소셜 로그인을 구현하며 작성한 글을 다시 읽어보며 발견한 사실이 있다.

<red>인가 코드 요청과 토큰 발급시 동일한 redirection_uri를 사용한다는 점</red>이다.

<br>

위에서 설명했듯이 현재 두레의 구글 소셜 로그인에서 사용하는 `redirect_uri`는 두가지이다.

1. 클라이언트에서 구글로 승인(인가코드 발급) 요청을 보낼 때
    1. 사용하고 있는 rediect_uri: `https:/www.doore.kro.kr/oauth2/code/google`
2. 백엔드에서 구글로 토큰 요청을 보낼 때
    1. 사용하고 있는 redirect_uri: `https://www.doore.kro.kr`
{: .box-yello}

백엔드에서 사용하고 있는 redirection uri를 프론트에서 사용하고 있는 redirectoin uri와 동일한 것으로 수정해주자…….

<br>
<br>
<br>
<br>

![해결](https://github.com/lcqff/lcqff.github.io/assets/71930280/5a89d699-a91a-4356-8c14-044e953194ba)

ㅠㅠ

<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/e3713eee-f1ab-4444-88d3-586fb94448d9" width="70%"/>

최종 포스팅에서는 결과만 요약해서 설명했지만

이 사이에 너무 의미없는 뻘짓이라 포스팅에 차마 담지 못한 삽질이 많다…. (현재 분량의 3배정도 된다)

그만큼 해결이 늦어져서 프론트분들에게 죄송할 따름이다ㅠㅠ

---

> 배운 점:
> 
> 1. 소셜 로그인은 프론트 개발 후에 작업하자 직접 클라이언트로 테스트해보자
> 2. 그래도 덕분이 이거 해결하면서 이후에 인프라 하며 경험할 삽질은 다 겪은거같다… +서브모듈과 프로파일에 대한 지식을 얻어 좋았다….
> 3. 지금은 소셜 로그인 코드가 그냥 쌩으로 작성되어있는데  Oauth2 라이브러리를 사용하는 것으로 리팩터링하자...