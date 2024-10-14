---
layout: post
title: "Spring Security(1)- OAuth2 구글 로그인 구현"
author: pam
categories: [SoundFlyer]
tags: [Spring, Spring Security] 
image: https://github.com/user-attachments/assets/19f7f0cf-3016-482b-9b4f-c27ebe0b78e1
description: "Spring Security를 사용한 OAuth2 로그인 원리를 학습하고 직접 구현합니다."
featured: true
toc: true
summary: "수동 인증 처리를 통한 Oauth2 로그인을 직접 구현해본적 있으나, 이러한 방식은 인증과 인가 방식에서 여러가지 불편함이 있었다. 
 해당 방식은 직접 인가코드를 호출하고, 토큰을 요청하고, 사용자 정보를 요청하는 번거로운 과정을 백엔드와 프론트상에서 수동으로 처리해야 했다. 
 또한 인가 처리가 아름답지 못하여 권한 처리를 할때마다 ‘내가 Spring Security를 알았으면!’하고 탄식했다.
 인증과 인가는 대부분의 어플리케이션에서의 필수 요소이기 때문에, 백엔드 개발자라면 Spring Security를 배워두는것이 좋겠다는 생각이 들었다."
---


---

<br>

# 개요

[수동 인증 처리를 통한 Oauth2 로그인](https://lcqff.github.io/kakao-login/)을 직접 구현해본적 있으나, 이러한 방식은 인증과 인가 방식에서 여러가지 불편함이 있었다.

해당 방식은 직접 인가코드를 호출하고, 토큰을 요청하고, 사용자 정보를 요청하는 번거로운 과정을 백엔드와 프론트상에서 수동으로 처리해야 했다. 
또한 인가 처리가 아름답지 못하여 권한 처리를 할때마다 ‘내가 Spring Security를 알았으면!’하고 탄식했다. 

인증과 인가는 대부분의 어플리케이션에서의 필수 요소이기 때문에, 백엔드 개발자라면 Spring Security를 배워두는것이 좋겠다는 생각이 들었다.

이번 게시글에서 다룰 내용은 아래와 같다.
> 구글 콘솔에서 프로젝트를 생성하는 방법을 알아본다.
> Spring Security를 사용한 Configuration 흐름을 알아본다.  
> Spring Security를 사용한 인증 처리 흐름을 알아본다.  
> Spring Security를 사용하여 실제 구글 Oauth2 로그인을 구현한다.  
> 여러 Oauth2 서비스가 동일한 코드를 사용할 수 있는 인터페이스를 구현한다.  

JWT 토큰 생성과 인가 처리와 같은 부분은 다른 포스팅으로 다룰 예정이다.

---

# (Google) 새 프로젝트 생성

[https://console.cloud.google.com](https://console.cloud.google.com/)에 접속하여 새로운 프로젝트를 생성하자.  
새 프로젝트를 생성한 뒤 `clientId`와 `secret`을 받아와야 한다.

## OAuth 동의화면

OAuth 동의화면에서 Scope설정을 할 수 있다.

![image](https://github.com/user-attachments/assets/c0b30a35-2424-4c77-b8d3-d3f02d44cd4e)
앱 정보는 필수항목만 작성한다.

![image](https://github.com/user-attachments/assets/1c6cf04b-1f80-4263-860d-2e252fa07e83)

위 두가지 항목만 요청한다.  
open Id같은 경우 `OpenID Connect`를 사용하기 위해서 필요한데, 나는 OIDC를 사용하지 않을거다

(OIDC는 이후에 학습해봐야겠다…)

## 사용자 인증 정보

![image](https://github.com/user-attachments/assets/3b3f8593-3a4c-4ba6-b24e-5eec6d3de6dd)

리다이렉션 URI는  Oauth2 요청을 보낼  백엔드 주소(`~/login/oauth2/code/{소셜 서비스 명}`)로 한다.

 <br>

> `/login/oauth2/code/{소셜 서비스 명}` 은 **Spring Security에서 기본적으로 제공해주는 리다이렉션Url**로, 다른 설정을 하지 않는다면 해당 엔드포인트를 로그인 요청으로 사용한다.
> 
> ![image](https://github.com/user-attachments/assets/ca223473-3421-4f22-8431-d056a8d42fd1)
> <cap>Oauth2LoginAuthenticationFilter에 기본 설정되어있다.</cap><br>
> 
> 위 redirection url에 의해 로그인이 성공하는 경우 백엔드 서버로 리다이렉션돼 토큰 발급 및 사용자 정보 요청이 진행되게 된다. 
> 
> (프론트 주소로 리다이렉션할 경우 토큰 발급과 사용자 정보 요청이 불가능해짐으로 **항상 백엔드로 redirection 되게 하자**.)  

  

Oauth2 로그인을 위해선 사용자 인증 정보 설정이 끝나고 발급받는 클라이언트 ID와 클라이언트 보안 비밀번호(Secret)가 필요하다.



# 설정파일 작성

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: {클라이언트 ID}
            client-secret: {클라이언트 보안 비밀번호}
            scope: profile, email
```

사용자 인증 정보에서 가져온 클라이언트 ID와 클라이언트 보안 비밀번호를 yml 파일에 입력해준다.   
민감 정보이기 때문에 외부에 노출되지 않도록 한다.

naver나 kakao의 설정파일 같은 경우 [**다른 포스팅**](https://lcqff.github.io/kakao-login/)을 참고하자.




# Spring Security Configuration 처리흐름

![image](https://github.com/user-attachments/assets/2e751f02-4cc8-4429-b2bd-c2e0b5d08820)

Spring Security의 설정과 초기화는 **SecurityConfigurer**의 `init()`과 `configure()`에서 일어난다.

위 두 메서드는 `HttpSecurity`의 `oauth2Login()` 메서드에 의해 실행되는데, oauth2Login()은 일반적으로 개발자가 직접 **`SecurityConfiguration`**과 같은 설정 클래스를 작성하는 것으로 실행된다.

## init()

```java

	@Override
	public void init(B http) throws Exception {
	
		// 인가코드 리다이렉션을 인터셉트하여 Access Token 및 사용자 정보를 요청하는 필더 OAuth2LoginAuthenticationFilter
		OAuth2LoginAuthenticationFilter authenticationFilter = new OAuth2LoginAuthenticationFilter(...);
		this.setAuthenticationFilter(authenticationFilter);
		
		// this.loginProcessingUrl = DEFAULT_FILTER_PROCESSES_URI = "/login/oauth2/code/*"
		// 인가 코드 요청이 Redirection되고, OAuth2LoginAuthenticationFilter가 인터셉트할 주소를 DEFAULT_FILTER_PROCESSES_URI로 설정한다.
		super.loginProcessingUrl(this.loginProcessingUrl); 

	 	....
		
		// 엑세스 토큰 교환용
		OAuth2AccessTokenResponseClient<OAuth2AuthorizationCodeGrantRequest> accessTokenResponseClient = getAccessTokenResponseClient();
		// 실제 사용자 정보 요청 처리용
		OAuth2UserService<OAuth2UserRequest, OAuth2User> oauth2UserService = getOAuth2UserService();
		
		// 실질적인 인증 처리용 (엑세스 토큰 요청, 사용자 정보 요청)
		OAuth2LoginAuthenticationProvider oauth2LoginAuthenticationProvider = new OAuth2LoginAuthenticationProvider(
				accessTokenResponseClient, oauth2UserService);
	
		 ....
	}
```

`init()`에서는 Spring Security에서 사용되는 인스턴스들을 초기화한다.

대표적으로 초기화 되는 인스턴스들은 아래와 같다. 

- **`OAuth2LoginAuthenticationFilter`**: 리다이렉션된 인가코드를 사용하여 `OAuth2LoginAuthenticationProvider`을 통해 Access Token을 요청하고 사용자 정보를 요청하는 전반적인 인증과정을 담당한다.
- **`OAuth2LoginAuthenticationProvider`**: accessToken을 요청하고 `OAuth2UserService`에 사용자 정보를 요청하여 OAuth2LoginAuthenticationFilter에 반환한다.
- **`OAuth2UserService`**: 사용자 정보을 제공한다.

## configure()

```java
@Override
	public void configure(B http) throws Exception {
	
		// OAuth2AuthorizationRequestRedirectFilter 초기화
		OAuth2AuthorizationRequestRedirectFilter authorizationRequestFilter;
		if (this.authorizationEndpointConfig.authorizationRequestResolver != null) {
			authorizationRequestFilter = new OAuth2AuthorizationRequestRedirectFilter(
					this.authorizationEndpointConfig.authorizationRequestResolver);
		}
		else {
		
		 // 인가코드의 발급 요청을 보내고, OAuth2AuthorizationRequestRedirectFilter가 인터셉트할 주소를 설정한다.
		 // 기본 주소: /oauth2/authorization/*
			String authorizationRequestBaseUri = this.authorizationEndpointConfig.authorizationRequestBaseUri;
			if (authorizationRequestBaseUri == null) {
				authorizationRequestBaseUri = OAuth2AuthorizationRequestRedirectFilter.DEFAULT_AUTHORIZATION_REQUEST_BASE_URI;
			}
			
			//커스터마이즈된 URI가 존재하는 경우 해당 URI를 사용한다.
			authorizationRequestFilter = new OAuth2AuthorizationRequestRedirectFilter(
					OAuth2ClientConfigurerUtils.getClientRegistrationRepository(this.getBuilder()),
					authorizationRequestBaseUri);
		}
	    ....
		
		// 인가 코드 요청이 Redirection되고, OAuth2LoginAuthenticationFilter가 인터셉트할 주소를 커스터마이즈 할 수 있다. 
		// init()에서 기본 url인 DEFAULT_FILTER_PROCESSES_URI로 설정되어 있다.
		// 기본 주소: /login/oauth2/code/*
		OAuth2LoginAuthenticationFilter authenticationFilter = this.getAuthenticationFilter();
		if (this.redirectionEndpointConfig.authorizationResponseBaseUri != null) {
			authenticationFilter.setFilterProcessesUrl(this.redirectionEndpointConfig.authorizationResponseBaseUri);
		}
		if (this.authorizationEndpointConfig.authorizationRequestRepository != null) {
			authenticationFilter
				.setAuthorizationRequestRepository(this.authorizationEndpointConfig.authorizationRequestRepository);
		}

		....
```

`configure()`에서는 인가코드 발급을 처리하는 `OAuth2AuthorizationRequestRedirectFilter`와  엑세스 토큰 요청 및 사용자 정보를 교환하는 `OAuth2LoginAuthenticationFilter`가 **인터셉트할 주소를 설정**한다.

Spring Security에서 인가코드 발급을 위해 사용되는 URI는 **`OAuth2AuthorizationRequestRedirectFilter`의** `DEFAULT_AUTHORIZATION_REQUEST_BASE_URI` 이다. 해당  상수는 */oauth2/authorization*라는 값으로 설정된다.

또한 인가코드 발급 후, 기본적으로 리다이렉션 되는 주소는 **`OAuth2LoginAuthenticationFilter`**의 `DEFAULT_FILTER_PROCESSES_URI`이다. 해당 상수는 */login/oauth2/code*로 설정된다.

`Oauth2LoginConfigure`는 configure() 메서드를 통해 만일 사용자가 **커스텀으로 작성한 인가코드 발급 URI과 Redirection URI가 존재한다면 해당 주소로 수정하고, 만일 존재하지 않는다면 기본주소를 사용하도록 설정한다**.



# Spring Security OAuth2 인증 처리 흐름

Sequence 다이어그램의 형태를 흉내내어 정리하긴 했지만 가독성 문제로 일부 클래스들은 생략된 상태라는점 참고해주길 바란다.

<br>

## 인가코드 발급

![image](https://github.com/user-attachments/assets/313c33ea-f99b-4fd4-a0da-c067d4a90160)

1. 로그인 버튼을 누르면 클라이언트는 */oauth2/authorization* 또는 커스텀된 인가코드 요청 uri로 요청을 보낸다.
2. */oauth2/authorization*로 오는 요청은 `OAuth2AuthorizationRequestRedirectFilter`에 의해 인터셉트 된다. 
3. `OAuth2AuthorizationRequestRedirectFilter`은 리다이렉션 URI(*/login/oauth2/code*)를 Query Parameter로 사용하여 Google에 인가 코드를 요청한다.

## 토큰 발급

![image](https://github.com/user-attachments/assets/4ef6889f-0b9e-4f6f-ab52-ca3c32126aeb)

1. Google은 **인가코드**를 Query Parameter로 사용하여 리다이렉션 URI */login/oauth2/code*로 리다이렉션 한다.
2. */login/oauth2/code*로 리다이렉션된 요청은 `OAuth2LoginAuthenticationFilter`에 의해 인터셉트 된다.
3. `OAuth2LoginAuthenticationProvider`가 인가코드를 사용하여 엑세스 토큰을 요청하고, 엑세스 토큰을 사용하여 사용자 정보를 요청한다.

## 사용자 정보 저장

![image](https://github.com/user-attachments/assets/19f7f0cf-3016-482b-9b4f-c27ebe0b78e1)

1. 사용자 정보를 요청받은 `Oauth2UserService`는 엑세스 토큰을 사용하여 Google에 사용자 정보를 요청한다.
2. `Oauth2UserService`가 Google에서 반환받은 사용자 정보를 OAuth2User 형태로 `OAuth2LoginAuthenticationProvider`에 반환한다.
3. `OAuth2LoginAuthenticationProvider`가 사용자 정보를 `OAuth2LoginAuthenticationFilter`에 반환한다. 이때 반환되는 OAuth2LoginAuthenticationToken은 **`Authentication`**의 구현체이다. 
4. **SecurityContext 내에Authentication 객체가 저장된다.**
5. LoginSuccessHandler가 실행된다.

<br>

# 코드

<br>

## Security Config

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class Oauth2ClientConfig {

    private final OAuth2UserService oAuth2UserService;
    private final LoginSuccessHandler loginSuccessHandler;

    @Bean
    SecurityFilterChain securityFilterChane(HttpSecurity http) throws Exception {
        // 사용자가 특정 URL에 접근할 수 있는 권한 설정
        http.authorizeHttpRequests(requests ->
                requests.requestMatchers("/**", "/api-docs/**", "/swagger-ui/**").permitAll()
        );

        http.csrf(AbstractHttpConfigurer::disable);

        http.oauth2Login(oauth2 -> oauth2
                .userInfoEndpoint(userInfoEndpointConfig ->
                        userInfoEndpointConfig.userService(oAuth2UserService))
                .successHandler(loginSuccessHandler)
                .failureUrl("http://localhost:3000")
        );

        return http.build();
    }
}
```

- **http.authorizeHttpRequests**: 사용자가 특정 URL에 접근할 수 있는 권한을 설정한다.
    - 로그인 없이도 조회가 가능한 사이트이기 때문에 모든 경로에 대한 접근을 `permitAll()`로 설정했다.
- **http.csrf().disable()**: CSRF 보호 설정을 disable한다.
    - RESTful API 에서는 따로 CSRF가 필요없다.
    - 클라이언트 요청에 대해서는 따로 JWT를 사용하여 CSRF 토큰을 대체한다. (다른 포스팅에서 작성)
- **oauth2Login**: Spring Security에서 사용하는 인스턴스를 초기화 및 커스텀 URI를 설정한다.
    - 사용자 정보 요청시 사용할 커스텀 클래스를 `oAuth2UserService`로 설정했다.
    - 로그인 성공시 커스텀한 `LoginSuccessHandler`가 실행되도록 설정했다.
    

## oAuth2UserService

![image](https://github.com/user-attachments/assets/ea0f3b13-28a2-4126-9602-3edfc59bd06a)


추후 다른 OAuth2 서비스가 추가되어도 코드 변경 없이 사용할 수 있도록 interface를 사용했다. 

- `ProviderUser`: 애플리케이션에서 필요한 사용자 속성을 정의한다.
- `OAuth2ProviderUser`: 추상 클래스로, 모든 OAuth2 서비스에서 동일한 형태로 제공되는 속성을 받아오는 메서드를 구현한다.
- `GoogleUser`: GoogleUser 객체를 저장하는 클래스로, OAuth2ProviderUser에서 구현되지 않은 메서드를 구현한다.

```java
@Slf4j
@Service
@Transactional
@RequiredArgsConstructor
public class OAuth2UserService extends DefaultOAuth2UserService {

    private final ArtistService artistService;

    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        ClientRegistration clientRegistration = userRequest.getClientRegistration();
        OAuth2User oAuth2User = super.loadUser(userRequest); //사용자 정보 반환
        ProviderUser providerUser = providerUser(clientRegistration, oAuth2User);
        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        artistService.register(registrationId, providerUser);
        log.info("회원가입 = " + providerUser.getName());
        return providerUser;
    }

    private ProviderUser providerUser(ClientRegistration clientRegistration, OAuth2User oAuth2User) {
        String registrationId = clientRegistration.getRegistrationId();
        if (registrationId.equals("google")) {
            return new GoogleUser(oAuth2User, clientRegistration);
        }
        return null; //todo: 예외 반환
    }
}

```

프론트에서  */oauth2/authorization* 로 인가코드 요청을 보내면, 사용자 정보 반환을 위하여 커스텀된 `OAuth2UserService`가 실행된다.

- `super.loadUser`: DefaultOAuth2UserService의 loadUser()을 실행시킨다.
    - 사용자 속성(attributes)과 사용자 권한(authorities)이 `OAuth2User` 형태로 리턴된다.
    - google에서 제공받는 attributes는 아래와 같다.
        
        > {sub= ..., name=임연후, given_name=연후, family_name=임, picture=https://..., email=yeonnu82g@gmail.com, email_verified=true}
        > 
- `providerUser(clientRegistration, oAuth2User)`: 제공받은 사용자 정보를 각 서비스에 맞는 객체(GoogleUser, NaberUser….) 로 변환시킨다.
- `artistService.register`: 제공받은 객체를 사용하여 실제 회원가입을 구현한다.

```java
public abstract class OAuth2ProviderUser implements ProviderUser {

    private final Map<String, Object> attributes;
    private final OAuth2User oAuth2User;
    private final ClientRegistration clientRegistration;

    public OAuth2ProviderUser(Map<String, Object> attributes, OAuth2User oAuth2User,
                              ClientRegistration clientRegistration) {
        this.attributes = attributes;
        this.oAuth2User = oAuth2User;
        this.clientRegistration = clientRegistration;
    }

    @Override
    public String getPassword() {
        return UUID.randomUUID().toString();
    }

    @Override
    public String getEmail() {
        return getAttributes().get("email").toString();
    }

    @Override
    public String getProvider() {
        return clientRegistration.getRegistrationId();
    }

    @Override
    public List<? extends GrantedAuthority> getAuthorities() {
        return oAuth2User.getAuthorities().stream()
                .map(authority -> new SimpleGrantedAuthority(authority.getAuthority()))
                .toList();
    }

    @Override
    public Map<String, Object> getAttributes() {
        return attributes;
    }
}

```

```java
public class GoogleUser extends OAuth2ProviderUser {

    public GoogleUser(OAuth2User oAuth2User, ClientRegistration clientRegistration) {
        super(oAuth2User.getAttributes(), oAuth2User, clientRegistration);
    }

    @Override
    public String getId() {
        return getAttributes().get("sub").toString();
    }

    @Override
    public String getName() {
        return getAttributes().get("name").toString();
    }

    @Override
    public String getPicture() {
        return getAttributes().get("picture").toString();
    }
}

```

이때 사용한 OAuth2ProviderUser와 GoogleUser는 위와 같다.

# 결과

![image](https://github.com/user-attachments/assets/a6263ba9-6d77-4ead-8eba-ac0142370922)

Spring Security를 통해 정상적으로 사용자 정보를 제공받았다.

다음에는 사용자의 uuid를 통해 JWT 토큰을 생성하고, JWT 토큰을 통해 사용자 요청을 검사하는 내용을 다뤄보겠다.


<br>

---

공부를 하겠다고 Spring Security 인강을 꽤 비싼 돈 주고 사긴 했는데 그 분량이 만만찮아서 아직 일부밖에 듣지 못했다...
인강 설명을 참고하긴 했지만 직접 디버깅을 통해 공부한 내용이라 게시글에서 틀린 부분이 존재할 수 있다.
혹시 틀린 부분이 있다면 댓글로 알려주시면 감사하겠습니다.