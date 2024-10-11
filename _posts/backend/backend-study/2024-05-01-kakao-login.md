---
layout: post
title: 카카오 로그인 api로 알아보는(1) -소셜 로그인
author: pam
categories: Backend-Study
tags: Spring Backend
sidebar:
---

## 개요

만들어보고 싶었던 어플리케이션을 제작하며 우선적으로 소셜 로그인 기능을 만들어보기로 했다.

개인적으로 동작이 궁금했고 공부해보고싶었던 분야이기도 하고, 권한은 전체적인 애플리케이션 기능에 사용될테니 차라리 미리 만들어두는게 좋겠다 생각했기 때문이다.

실제 코드 작성에 앞서 로그인은 어떤 방식으로 이루어지며, 그중 소셜 로그인은 또 어떻게 다른지 알아보도록 하자~

## HTTP 프로토콜: 왜 사용자 정보를 저장해야할까?
### Statuful 통신의 한계

<div class="callout">
  <div>💡</div>
  <div>
	<strong>stateful(상태 유지)</strong><br/>
	서버가 클라이언트의 상태를 보존하는 것
  </div>
</div>


예를 들어 쿠키나 세션을 통해 서버에 클라이언트 인증 정보를 저장하여 클라이언트가 다른 페이지로 이동하더라도 로그인 상태가 유지되는 것을 **stateful**하다고 한다.

- 서버에 클라이언트의 상태가 저장되는 만큼 statuful 통신은 서버의 부하가 크다. 
  - 서버가 1만명의 클라이언트 정보를 저장할 수 있다면, 1만 명 이상이 연결될 경우 1만 번째 이후의 클라이언트들은 이미 연결된 클라이언트가 빠져나가고 나서야 처리될 수 있다.
- 클라이언트의 정보를 저장하고 있던 기존 서버에 장애가 생겨 다른 서버에 연결되는 경우 클라이언트의 정보를 다시 저장하는 문제가 있다.

### Stateless 통신

<div class="callout">
  <div>💡</div>
  <div>
	<strong>Stateless(무상태)</strong><br/>
  서버에 클라이언트의 상태가 보존되지 않는다.
  </div>
</div>

stateless 통신에서 서버는 요청이 들어오면 응답을 보내는 역할만 수행하고, **상태관리를 위한 책임은 모두 클라이언트**에게 있다. 클라이언트는 상태 관리를 위해 서버와의 통신에서 상태 정보 데이터를 담아 전달한다.

- 서버에서 클라이언트의 상태를 저장하지 않으므로 기존 서버에 장애가 생겨 서버가 바뀌더라도 응답에 문제가 없다.
- 대량의 트래픽이 발생하더라도 각각의 요청이 독립적으로 처리되므로 서버 확장을 통한 병렬적 처리로 수월하게 해결할 수 있다.

<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/591708e2-0ec2-47de-8339-070f4a12d4b1">
  여러 서버에서 트래픽이 독립적으로 처리된다. 
</div>

### HTTP 프로토콜

HTTP 프로토콜은 `Stateless`한 특징을 지닌다. 따라서 로그인을 유지하기 위해선 **모든 요청에 클라이언트의 정보를 담아 보내는 과정**이 필요하다.

결국 로그인 상태의 유무는 *요청에 사용자를 인증할 수 있는 정보가 담겨져 있는가*로 나누어진다. 즉슨, 사용자 정보의 상태 유지를 통해 로그인을 유지할 수 있다.

그러나 이러한 상태 유지를 위해 매번 파라미터에 사용자 정보를 넘겨주는 작업은 번거롭다. <blue>HTTP 프로토콜에서 상태를 유지하기 위해 제공하는 메커니즘</blue>들을 알아보자.


## 로그인 방법

로그인이란 것은 유지가 되어야 의미가 있다 (당연히, 로그인 페이지를 벗어났을 때 로그인이 풀리면 의미가 없으니까) 로그인을 한다는 것은 사용자 정보가 상태 유지된다는 의미다.

http 통신에서 이러한 상태유지를 위해 사용하는 메커니즘들을 간단히 알아보자.

실제로 사용할 JWT와 OAuth에 대해 자세히 알아볼 것이므로 쿠키와 세션은 개념만 짚고 넘어가겠다. 


### 쿠키

- 키와 값으로 구성된 클라이언트 상태 정보 데이터 파일로 **클라이언트 로컬에 저장**된다.
- 사용자가 요청하지 않아도 브라우저가 알아서 요청의 Request Header에 쿠키를 삽입한다.
- 그러나 쿠키값은 변조가 가능하며, <red>민감 정보가 노출</red>된다는 단점이 있다.
    - ex) 민감 정보가 쿠키에 저장될 경우 로컬 pc와 클라이언트→서버 네트워크에 노출된다.
    - ex) memberId=1이라는 쿠키로 로그인을 유지하는 경우, 쿠키값을 변조하는 것으로 타인의 계정에 로그인할 수 있다.

 

### 세션

쿠키의 노출 이슈를 보안하기 위해 나온 대안이다. 

![세션](https://github.com/lcqff/lcqff.github.io/assets/71930280/4c4ca588-470c-441a-b3a5-67512015ce04)

- 민감 정보가 노출되지 않도록 **서버에 저장**하는 방법이다.
- 쿠키에 사용자 정보가 아닌 **세션 ID**를 저장한다.
- 서버에서 sessionId를 통해 사용자 정보를 찾을 수 있다.
- 쿠키값(SessionId)가 노출되더라도 중요한 정보가 들어있지 않다.

### OAuth

<div class="callout">
  <div>💡</div>
  <div>
	<strong>Oauth(Open Authorization)</strong><br/>
	인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단
  </div>
</div>

사용자가 어떤 애플리케이션에 정보를 직접 제공하는 대신에 카카오톡이나 구글과 같은 **다른 웹사이트에 있는 자신의 정보를 접근**할 수 있도록 하는 것이다.

![OAuth](https://github.com/lcqff/lcqff.github.io/assets/71930280/1dc3f76b-0e06-40e2-a6bd-6ae887980f8f)

- **자원 서버(Resource Server or Service Server)** : Client가 제어하고자 하는 자원 보유하고 있는 서버 (클라이언트가 실제로 이용하고자 하는 서버)
- **인증 서버(Authorization Server)** : 클라이언트가 자원 서버의 서비스를 이용할 수 있게 인증하고 토큰을 발생해주는 서버 ex. 구글, 페이스북 등

OAuth은 로그인 방법이라기 보다는 말 그대로 **사용자 정보를 제공해주는 수단**이다. 클라이언트는 Access Token을 통해 인증서버로부터 사용자 정보를 요청할 수 있다.

일반적로 JWT와 함께 사용된다. 이론만 봐서는 session과 함께 사용할수도 있을 것 같지만 따로 저장소를 둘 필요가 없는 JWT가 더 선호되나보다🤔 (JWT에 대해서는 다음 게시글에서 자세히 살펴본다.) 


## OAuth: 카카오 로그인

OAuth는 **`인가 코드 발급 → Access Token 발급 → 사용자 정보 제공`**의 3단계로 이루어진다. 실제 카카오 로그인 기능 구현을 해보며 각 단계별 내용을 알아보자. 

[카카오 로그인 문서(Rest api)](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#propertykeys)

### 0. 설정

카카오 로그인을 사용하기 위해선 코드 작성에 앞서 카카오 developer에 앱을 생성하고, 설정하는 과정이 필요하다. (이 부분을 잘못해서 문제가 자주 발생함…)

그러나 [카카오 로그인 문서](https://developers.kakao.com/docs/latest/ko/kakaologin/prerequisite)의 설정하기 페이지에 앞서 필요한 설정들이 상세히 설명돼있다. 따라서 해당 글에서 설명하진 않을 것… [이 블로그 게시글](https://velog.io/@appti/Spring-Security-OAuth2-%EC%B9%B4%EC%B9%B4%EC%98%A4)도 많은 도움이 됐다.

<br>

![동의항목](https://github.com/lcqff/lcqff.github.io/assets/71930280/cdd9c86a-948b-4f44-ac15-def37c18fbe3)

이 내용은 단순히 참고용으로 작성해두는데, 나의 경우에는 닉네임과 프로필 사진 정보를 필수동의가 아니라 ‘선택 동의’로 설정할 경우 로그인시 동의화면 페이지가 뜨지 않고 사용자 정보에서 profile이 null로 넘어왔다.

필수동의로 설정을 변경하자 동의화면 페이지도 정상적으로 뜨고 profile 내용도 잘 채워져 전달되었다.

<br>

추가적으로, 편의를 위해 Spring Security의 **[OAuth2](https://docs.spring.io/spring-security/reference/servlet/oauth2/index.html)를** 사용했다. 

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          kakao:
            client-id: my-client-id
            client-secret: my-client-secret
            redirect-uri: my-redirect-uri
            client-name: Kakao
            scope:
              - profile_nickname
              - profile_image
        provider:
          kakao:
            authorization-uri: https://kauth.kakao.com/oauth/authorize
            token-uri: https://kauth.kakao.com/oauth/token
            user-info-uri: https://kapi.kakao.com/v2/user/me
            user-name-attribute: id
```

`Spring Security OAuth2`에서 provider로 kakao를 제공하지 않기 때문에 provider직접 작성해야한다.

### 1. 인가 코드 발급

![인가코드 받기](https://github.com/lcqff/lcqff.github.io/assets/71930280/e73997c2-7018-4954-a34e-8f167148ec03)

사용자 동의(인가)를 구하는 과정이다. 인가 코드 발급에 성공하면 발급받은 코드를 통해 토큰을 발급 받을 수 있다.

#### 기본정보

**URI**

| 메서드 | URL | 인증 방식 |
| --- | --- | --- |
| GET | https://kauth.kakao.com/oauth/authorize | - |

**쿼리 파라미터**

| 이름 | 타입 | 설명 | 필수 |
| --- | --- | --- | --- |
| client_id | String | 앱 REST API 키[내 애플리케이션] > [앱 키]에서 확인 가능 | O |
| redirect_uri | String | 인가 코드를 전달받을 서비스 서버의 URI[내 애플리케이션] > [카카오 로그인] > [Redirect URI]에서 등록 | O |
| response_type | String | code로 고정 | O |

{: .box-yello}
- **client_id**: 앱을 생성할때 발급받은 `REST_API_KEY`
- **redirect_uri**: 인가 코드 발급이 성공한 후 해당 uri를 자동 호출한다.

이 외 필수가 아닌 파라미터는 kakao developer 문서를 참고한다.

#### 코드

```dart
Future<void> _signInWithKakao() async {
    try {
      final clientState = Uuid().v4();
      final url = Uri.https('kauth.kakao.com', '/oauth/authorize',
        {
          'response_type': 'code',
          'client_id': 'rest api key 작성',
          'redirect_uri': 'http://localhost:8080/login/oauth2/code/kakao',
          'state': clientState,
        });

      final result = await FlutterWebAuth.authenticate(
          url: url.toString(), callbackUrlScheme: "webauthcallback"
      );

      final body = Uri.parse(result).queryParameters;

      print(body);
    } catch (error) {
      print('카카오톡 로그인 실패 $error');
    }
  }
```
flutter


프론트 프레임워크로 flutter를 사용하고 있다. 

로그인 버튼을 클릭했을 경우 `_signInWithKakao()`함수가 실행되도록 프론트 코드를 작성했다.

(프론트에서 직접 외부 api를 호출하지 않고 백엔드에서 호출하도록 할까 싶었지만 그러면 api 호출 횟수가 1회 늘어나는 꼴이 되는지라... 그냥 프론트에서 처리하기로 했다.)

client_id 같은 경우는 외부에 노출되어선 안되는 정보기 때문에 env파일 같은 곳에 저장해두기로 하자.

<br>

`http://localhost:8080/login/oauth2/code/kakao`은 토큰 발급용 내부 api uri로, 인가코드 발급이 종료되면 바로 내부 api로 리다이렉트 되도록 했다.

> 인가 코드 받기 요청의 응답은 HTTP 302 리다이렉트되어, `redirect_uri`에 `GET` 요청으로 전달됩니다.

### 2. 토큰 발급
<img width="779" alt="토큰 받기" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/260b613a-7097-4709-987e-50666ecf57da">

#### 기본 정보

**URI**

| 메서드 | URL | 인증 방식 |
| --- | --- | --- |
| POST | https://kauth.kakao.com/oauth/token | - |

**쿼리 파라미터**

| 이름 | 타입 | 설명 | 필수 |
| --- | --- | --- | --- |
| grant_type | String | authorization_code로 고정 | O |
| client_id | String | 앱 REST API 키[내 애플리케이션] > [앱 키]에서 확인 가능 | O |
| redirect_uri | String | 인가 코드가 리다이렉트된 URI | O |
| code | String | 인가 코드 받기 요청으로 얻은 인가 코드 | O |
| client_secret | String | 토큰 발급 시, 보안을 강화하기 위해 추가 확인하는 코드[내 애플리케이션] > [보안]에서 설정 가능ON 상태인 경우 필수 설정해야 함 | X |

인가 코드 받기 요청의 응답은 `redirect_uri`에 `code` 파라미터로 전달된다. 

redirect된 api에서 토큰 발급 api를 호출하는 것으로 토큰을 발급받을 수 있다. 

**응답**

| 이름 | 타입 | 설명 | 필수 |
| --- | --- | --- | --- |
| token_type | String | 토큰 타입, bearer로 고정 | O |
| access_token | String | 갱신된 사용자 액세스 토큰 값 | O |
| id_token | String | ID 토큰 값 | X |
| expires_in | Integer | 액세스 토큰 만료 시간(초) | O |
| refresh_token | String | 갱신된 사용자 리프레시 토큰 값, 기존 리프레시 토큰의 유효기간이 1개월 미만인 경우에만 갱신 | X |
| refresh_token_expires_in | Integer | 리프레시 토큰 만료 시간(초) | X |

<br>

<div class="captionedImg">
  <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/ff286312-78f6-4059-af48-e740adfadd93">
  토큰을 받아오는 과정 
</div>


#### 코드

```java
@GetMapping("/oauth2/code/kakao")
public ResponseEntity<LoginResponse> getKakaoToken(@RequestParam("code") String code) {
		return ResponseEntity.status(HttpStatus.OK).body(loginService.getKakaoToken(code));
}
```
controller

Code도 DTO를 통해 받아올까 생각했는데 어차피 외부 api의 응답값이고, 스펙이 바뀔일은 없을 것 같아서 그냥 파라미터로 받아왔다.

```java
public LoginResponse getKakaoToken(String code) {
        KakaoTokenResponse tokenResponse = tokenExchanger.getToken(code);
        //...
}
```
service

`KakaoTokenResponse` DTO에는 accessToken과 expiresIn 필드가 존재한다.  expiresIn은 사용하진 않았는데 일단 넣어둠…

```java
@Component
@RequiredArgsConstructor
public class TokenExchanger {
    private final WebClient webClient;
    private static final String GRANT_TYPE = "authorization_code";
  
    @Value("spring.security.oauth2.client.provider.kakao.token-uri")
    private String KAKAO_REQUEST_URI;
    
    @Value("${spring.security.oauth2.client.registration.kakao.client-id}")
    private String KAKAO_CLIENT_ID;

    @Value("${spring.security.oauth2.client.registration.kakao.redirect-uri}")
    private String KAKAO_REDIRECT_URI;

    public KakaoTokenResponse getToken(String code) {
        String request_uri =
                KAKAO_REQUEST_URI + "?grant_type=" + GRANT_TYPE + "&client_id=" + KAKAO_CLIENT_ID + "&redirect_uri="
                        + KAKAO_REDIRECT_URI + "&code=" + code;
        Flux<KakaoTokenResponse> response = webClient.post()
                .uri(request_uri)
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .retrieve()
                .bodyToFlux(KakaoTokenResponse.class);

        return response.blockFirst();
    }
}
```
utils

토큰 발급 외부 api를 호출하는 코드이다. 

<red>*주의</red>: Secret값을 사용하고 있지 않은지 확인하자. 사용하고 있다면 uri에 Secret값을 추가해야한다.  

<br>

외부 api 호출을 위해 **webClient**를 사용했다. webClient 대신 RestTemplate을 사용할 수도 있는데 webClient가 더 좋아보여서 webClient로... 
RestTemplate과 WebClient는 나중에 자세히 비교해서 글을 써보려 한다.

{: .box-error}
'org.springframework.web.reactive.function.client.WebClient' that could not be found.

참고로 webClient사용시 위와 같은 오류가 발생할 수 있는데, WebClient에 Bean이 할당되지 못해 발생하는 문제로 아래 설정을 통해 해결할 수있다.

```java
@Configuration
public class WebConfig {
    @Bean
    WebClient webClient(WebClient.Builder builder) {
        return builder.build();
    }
}
```

<br>

위 코드를 통해 제공받은 access token은 사용자에게 제공한 token의 일치 여부만 판별할 수 있을 뿐 **token 내에 사용자 정보가 담겨있진 않다**. 

따라서 token을 통해 사용자를 식별하기 위해서는 사용자 정보 가져오기 api를 호출해 사용자 정보를 획득하거나, 사용자 정보를 가져온 다음 session과 같이 저장소에 저장된 사용자 정보를 확인하기 위한 key로 사용하는 수밖애 없다.

반면 **JWT는 token 그 자체가 JSON 형식의 정보**를 담고 있으며  디코딩을 통해 사용자 정보를 취득 가능하다. JWT에 대해서는 다음 게시글에서 자세히 알아보겠다.

### 3. 사용자 정보 가져오기

<img width="790" alt="로그인 처리" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/7eff638e-631b-4d4e-b2f6-7bf0c4d36e9c">

#### 기본 정보

**uri**

| 메서드 | URL | 인증 방식 |
| --- | --- | --- |
| GET/POST | https://kapi.kakao.com/v2/user/me | 액세스 토큰서비스 앱 어드민 키 |

**요청 헤더**

| 이름 | 설명 | 필수 |
| --- | --- | --- |
| Authorization | Authorization: Bearer ${ACCESS_TOKEN} 인증 방식, 액세스 토큰으로 인증 요청 | O |
| Content-type | Content-type: application/x-www-form-urlencoded;charset=utf-8요청 데이터 타입 | O |

**응답**

```json
HTTP/1.1 200 OK
{
    "id":123456789,
    "connected_at": "2022-04-11T01:45:28Z",
    "kakao_account": { 
        "profile_nickname_needs_agreement	": false,
        "profile_image_needs_agreement	": false,
        "profile": {
            "nickname": "홍길동",
            "thumbnail_image_url": "http://yyy.kakao.com/.../img_110x110.jpg",
            "profile_image_url": "http://yyy.kakao.com/dn/.../img_640x640.jpg",
            "is_default_image":false,
            "is_default_nickname": false
        },
      //...
    },
		//...
}
```

내가 사용하지 않을 응답값은 생략해두었다. 자세한 응답값은 문서를 참고하자. 

#### 코드

```java
    public LoginResponse getKakaoToken(String code) {
        KakaoTokenResponse tokenResponse = tokenExchanger.getToken(code);
        KakaoUserInfo userInfo = userInfoFetcher.getKaKaoUserInfo(tokenResponse.accessToken());
				//사용자 조회
        //jwt 토큰 생성
        return new LoginResponse(user.getId(),"temp Token");
    }
```
service

카카오 서버에 저장된 유저 정보를 가져오는 코드를 추가했다. 
이후  이 정보를 기반으로 JWT 토큰을 생성할 것이다.

```java
@Component
@RequiredArgsConstructor
public class UserInfoFetcher {
    private final WebClient webClient;

    @Value("spring.security.oauth2.client.provider.kakao.user-info-uri")
    private String KAKAO_REQUEST_URI;

    public KakaoUserInfo getKaKaoUserInfo(String token) {
        Flux<KakaoUserInfo> response = webClient.post()
                .uri(KAKAO_REQUEST_URI)
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .header("Authorization", "Bearer " + token)
                .retrieve()
                .bodyToFlux(KakaoUserInfo.class);
        return response.blockFirst();
    }
}
```
utils

```java
public record KakaoUserInfo(
    Long id,

    @JsonProperty("kakao_account")
    KakaoAccount kakaoAccount
) {
    public record KakaoAccount(
            @JsonProperty("profile_nickname_needs_agreement")
            Boolean profileNicknameNeedsAgreement,

            @JsonProperty("profile_image_needs_agreement")
            Boolean profileImageNeedsAgreement,
            Profile profile,
            String email,

            @JsonProperty("email_needs_agreement")
            Boolean emailNeedsAgreement,

            @JsonProperty("is_email_valid")
            Boolean isEmailValid,

            @JsonProperty("is_email_verified")
            Boolean isEmailVerified
    ){
        public record Profile(
                String nickname,

                @JsonProperty("profile_image_url")
                String profileImageUrl,

                @JsonProperty("is_default_image")
                Boolean isDefaultImage,

                @JsonProperty("is_default_nickname")
                Boolean isDefaultNickname
        ) {}
    }
}
```
dto


위에서 서술했 듯 ‘동의항목’을 선택으로 설정해두었더니 로그인시 동의항목 화면이 뜨지 않고 바로 Profile이 null로 전달되는 문제가 있었다;
필수로 변경하였더니 정상적으로 값이 입력되었다. 🤔

![성공](https://github.com/lcqff/lcqff.github.io/assets/71930280/aff60774-2068-4d25-b67f-589edd9b82b3)

카카오 정보 받아오기 성공

이제 받아온 정보를 가지고 JWT를 생성하는 단계만 남았다.

JWT에 대한 정보와 그 과정은 다음 게시글에서…..😄

## 고민거리들

**~~Cors 에러~~**

음? 분명 외부 api를 불러오는 과정에서 Cors 오류가 발생했었는데 갑자기 확장 프로그램을 꺼도 정상 작동한다🤔

Cors가 아니라 코드 문제였을까….? 그건 그렇고 외부 api를 호출하면 당연히 Cors가 발생해야할 것 같은데 발생하지 않는 것도 신기하다.

<br>

**소셜 로그인 확장**

현 애플리케이션은 Kakao뿐만 아니라 Google 소셜 로그인 또한 가능하도록 계획돼있다.

DTO는 각 소셜 로그인이 제공하는 응답 형태가 다르니 어쩔수 없다 쳐도, 다른 소셜 로그인이 추가될때마다 동일한 기능을 하는 메소드를 각 소셜 로그인 별로 추가하는 건 너무 별로다. 구글 로그인을 추가하면서 한 번 방법을 찾아봐야겠다…

---

<br>

> 배운 점: 소셜 로그인 기능 하나를 구현하면서 정말 시행착오가 많았다… 특히 코드가 아니라 소셜 로그인 설정에서 생긴 문제가 많았다. (flutter는 할많하않) <br>
> 그래도 로그인 원리와 그 과정을 알게되니 정말 좋다🥹
