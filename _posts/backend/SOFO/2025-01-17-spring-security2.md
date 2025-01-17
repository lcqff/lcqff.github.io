---
layout: post
title: "Spring Security(2)- JWT"
author: pam
categories: [SOFO]
tags: [Spring, Spring Security] 
image: https://github.com/user-attachments/assets/b69b84ea-2b36-45f0-aa3a-7bd83bd10cf6
description: "JWT를 사용하여 사용자 인증 정보를 Statless하게 관리합니다."
featured: true
toc: true
summary: "지난 게시글에서는 Spring Security를 사용하여 OAuth2 로그인을 구현하고, 이를 통해 로그인한 사용자 정보를 Stateful하게 관리했다.

이번 게시글에서는 Stateful하게 관리하던 인증 정보를 JWT 토큰을 사용하여 Stateless하게 변경할 것이다."
---


---

<br>

> 지난 게시글  
> [**Spring Security(1)- OAuth2 구글 로그인 구현**](https://lcqff.github.io/spring-security1/)

---
<br>

지난 게시글에서는 Spring Security를 사용하여 OAuth2 로그인을 구현하고, 이를 통해 로그인한 사용자 정보를 Stateful하게 관리했다.

이번 게시글에서는 Stateful하게 관리하던 인증 정보를 JWT 토큰을 사용하여 Stateless하게 변경할 것이다.

<br>

# Stateful한 인증 정보 관리

클라이언트의 인증 상태를 Stateful하게 관리하는 경우, 서버는 세션 ID를 사용하여 클라이언트의 인증 상태를 기억하는 책임을 가지게 된다.

그러나 이런 Stateful한 방식은 아래와 같은 단점을 지니고 있다.

- 서버 재배포 시 사용자 인증 정보가 사라진다.
- 서버가 여러대인 경우 사용자 인증 정보를 서버간 공유하는 과정이 필요하다. (서버 확장성이 좋지 않다.)

따라서 일반적으로는 Stateless한 인증 정보 관리가 선호된다. 

또한 **RESTful API**를 개발하는 만큼 stateless한 인증 관리가 일반적으로 사용된다. (REST 원칙 중 하나인 무상태성을 기억하라…)

<br>

# Stateless한 인증 정보 관리 - JWT

따라서 나는 Stateless한 인증 정보 관리를 위하여 JWT를 사용할 것이다.

생성한 토큰은 클라이언트에 저장되고, 이후 클라이언트는 서버에 요청을 보낼 때 저장된 토큰을 헤더에 담아 보내는 것으로 사용자 인증 정보를 서버에 전달할 수 있다.

이를 위해 서버가 해야할 역할은 아래와 같다.

{: .box-note}
1. 사용자 로그인 시 JWT 토큰 생성
2. 클라이언트에 토큰 저장
3. 클라이언트 요청시 요청 헤더에서 토큰 확인
4. 토큰 해석하여 인증 정보 확인
5. SecurityContext에 사용자 인증 정보 저장

<br>

## Securityconfig 파일 수정

```java
http.sessionManagement(sessionManagement -> sessionManagement.sessionCreationPolicy(STATELESS));
http.addFilterBefore(jwtAuthorizationFilter, UsernamePasswordAuthenticationFilter.class);
```

1. 더이상 Stateful한 인증 정보 관리를 사용하지 않으니 `sessionManagement`에서 세션 정책을 **STATELESS**로 설정해주었다.
2. `UsernamePasswordAuthenticationFilter` 전에 `jwtAuthorizationFilter`가 삽입되도록 필터 체인 순서를 변경하였다. 

이때 `UsernamePasswordAuthenticationFilter`은 Oauth2에서는 사용되지 않는, 폼 기반 로그인에서 사용되는 필터이다.  사용자 비밀번호 기반의 인증 처리를 담당하며, UsernamePasswordAuthenticationToken을 생성하여 Authentication을 반환하는 식으로 인증을 처리한다.

우리도 `jwtAuthorizationFilter`을 사용하여 Authentication을 생성 및 반환하는 방식으로 인증을 진행할 것이기에 `UsernamePasswordAuthenticationFilter`의 앞에 넣어주도록 하자.

```
Security filter chain: [
  DisableEncodeUrlFilter
  WebAsyncManagerIntegrationFilter
  SecurityContextHolderFilter
  HeaderWriterFilter
  CorsFilter
  LogoutFilter
  OAuth2AuthorizationRequestRedirectFilter
  OAuth2LoginAuthenticationFilter
  JwtAuthorizationFilter
  RequestCacheAwareFilter
  SecurityContextHolderAwareRequestFilter
  AnonymousAuthenticationFilter
  SessionManagementFilter
  ExceptionTranslationFilter
  AuthorizationFilter
]
```

위 두 설정을 추가하고 애플리케이션을 돌려보면 이전엔 없었던 SessionManagementFilter와 JwtAuthorizationFilter가 생성된 걸 확인할 수 있다.


<br>

## 토큰 생성

클라이언트가 사용자 인증 정보를 전달하기 위해선, 먼저 서버에서 JWT를 생성해야한다.

이때 생성한 JWT를 클라이언트의 Cookie에 저장하는 것으로 클라이언트가 이를 참조해 전달할 수 있게 한다.

```java
@Slf4j
@Component
@Transactional
@RequiredArgsConstructor
public class LoginSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
    private final CookieService cookieService;
    
    @Value(value = "${server.front}")
    private String domain;
    
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
                                        Authentication authentication) throws IOException {

        OAuth2ProviderUser user = (OAuth2ProviderUser) authentication.getPrincipal();

        String uuid = user.getUuid();
        List<? extends GrantedAuthority> role = user.getAuthorities();

        cookieService.authenticate(uuid, role, response);

        String url = UriComponentsBuilder.fromUriString(domain)
                .path("")
                .build()
                .toUriString();

        response.sendRedirect(url);
    }
}

```

`LoginSuccessHandler`에 코드를 추가해 로그인 성공 시 Uuid와 Role을 사용하여 JWT를 만들어 쿠키에 저장하도록 하자.

```java
@Service
public class JwtService {

    @Value("${jwt.expiration_time}")
    private long ACCESS_TOKEN_EXP_TIME;
    @Value("${jwt.secret}")
    private String SECRET;

    public String createToken(String uuid, List<? extends GrantedAuthority> authorities) {
        List<String> roles = authorities.stream()
                .map(GrantedAuthority::getAuthority) // 권한을 String으로 변환
                .toList();
        Map<String, Object> claims = new HashMap<>();
        claims.put("authority", roles);
        ZonedDateTime now = ZonedDateTime.now();
        ZonedDateTime tokenValidity = now.plusSeconds(ACCESS_TOKEN_EXP_TIME);

        return Jwts.builder()
                .setClaims(claims) // setClaims와 setSubject 순서 지킬것
                .setSubject(uuid)
                .setIssuedAt(Date.from(Instant.now()))
                .setExpiration(Date.from(tokenValidity.toInstant()))
                .signWith(Keys.hmacShaKeyFor(SECRET.getBytes(StandardCharsets.UTF_8)), SignatureAlgorithm.HS256)
                .compact();
    }

    public Claims validateToken(String token) {
        try {
            return Jwts.parserBuilder()
                    .setSigningKey(Keys.hmacShaKeyFor(SECRET.getBytes(StandardCharsets.UTF_8)))
                    .build()
                    .parseClaimsJws(token)
                    .getBody();
        } catch (JwtException e) {
            throw new JwtException("유효하지 않은 토큰입니다.");
        }
    }
}
```

JWT를 생성하는 코드에 대한 설명은 생략한다.

```java
    private Cookie makeCookie(String key, String value, int maxAge) {
        Cookie cookie = new Cookie(key, value);
        cookie.setSecure(true); 
        cookie.setHttpOnly(true);
        cookie.setPath("/");
        cookie.setMaxAge(maxAge);

        return cookie;
    }

```

이후 생성된 JWT(value)을 사용하여 쿠키를 생성한다.

JWT가 사용자의 인증정보를 지니고 있고, 서버에서 이를 통해 사용자를 분간하는 만큼 JWT는 매우 민감한 정보이다. 이를테만, 해커가 내 ID나 비밀번호를 알지 못해도 내 JWT 값을 탈취한다면 내 계정에 로그인 할 수 있다.

따라서 JWT를 보관하는 Cookie에 최소한의 **보안 설정**을 걸어두도록 하자.

{: .box-note}
- cookie.setSecure(true): HTTPS 연결에서만(암호화된 네트워크에서만) 쿠키가 클라이언트로 전송되도록 한다.
- cookie.setHttpOnly(true): JavaScript에서 쿠키에 접근하지 못하도록 한다. **(XSS 방어)**

사용자가 웹페이지를 읽을 때, HTML과 JS 파일이 클라이언트로 전달된다. 

이때 전달된 JS 파일은 서버에 저장되어 있으나, 실제 실행은 클라이언트에서 이루어진다.

따라서 어떤 악성 사용자가 게시글에 쿠키를 탈취하는 JavaScript 코드를 삽입하고, 이것이 검열되지 않고 서버에 저장되는 경우 그것을 READ해온 사용자의 클라이언트 환경에서 JavaScript 코드가 실행된다. 즉, 해커에게 사용자의 쿠키(인증 정보)가 전달된다. 이것이 XSS 공격이다.

Cookie 설정을 통해 XSS를 방어하되 애초에 JavaScript 삽입 공격이 발생하지 않도록 Secure Coding 하도록 하자… (이것도 공부해야됨…)

<br>

## JwtAuthorizationFilter 작성

위에서 우린 사용자 인증 정보를 기반으로 JWT 토큰을 생성하고, 이를 클라이언트에 전달하였다.

이제 클라이언트는 요청 헤더에 JWT를 담아 서버로 전달하고, 우리는 요청 헤더에서 JWT 토큰을 찾아 해석하여 인증 정보를 확인하면 된다.

`JwtAuthorizationFilter`은 JWT 토큰을 사용한 인가 및 인증을 담당한다. 

{: .box-note}
1. 클라이언트 요청 시 요청 헤더에서 토큰 확인
2. 토큰 해석하여 인증 정보 확인
3. SecurityContext에 사용자 인증 정보 및 권한 저장

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class JwtAuthorizationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            filterChain.doFilter(request, response); //다음 필터로 전달
            return;
        }

        String token = header.replace("Bearer ", "");
        Claims claims = jwtService.validateToken(token);

        String uuid = claims.getSubject();
        List<?> rawAuthority = (List<?>) claims.get("authority");
        List<? extends GrantedAuthority> authorities = rawAuthority.stream()
                .map(auth -> new SimpleGrantedAuthority(auth.toString()))
                .toList();
        Authentication authentication = new UsernamePasswordAuthenticationToken(uuid, null, authorities);
        SecurityContextHolder.getContext().setAuthentication(authentication);

        filterChain.doFilter(request, response); //다음 필터로 전달
    }
}

```

- 만일 토큰 형식이 유효하지 않거나 비어있으면 `SecurityContext`에 값을 저장하지 않는다.
- 토큰 형식이 유효한 경우 토큰을 해석한다.
- 토큰에 저장된 Claim을 사용하여 `Authentication` 생성
    - `UsernamePasswordAuthenticationToken`을 통해 사용자 인증 정보 저장
- `SecurityContext`에 `Authentication`을 저장한다.

<br>

# 동작 확인

![Image](https://github.com/user-attachments/assets/b69b84ea-2b36-45f0-aa3a-7bd83bd10cf6)

로컬서버에서 로그인 후 클라이언트에 쿠키가 저장되었나 확인해보았다. 
잘 저장되었음을 확인할 수 있다. 보안 설정도 잘 설정되어있다.

(Secure한 Cookie라서 http://localhost 환경에서도 저장 안될 줄 알았는데, 확인해보니 로컬호스트에선 예외적으로 쿠키 전송이 허용되는 모양이다.)

<br>

# +) Cors 오류

![Image](https://github.com/user-attachments/assets/5d58a860-dad2-4594-b34a-437dd6e088f4)
![Image](https://github.com/user-attachments/assets/1999de67-1c2c-4b9b-86de-70c7aa0ec6c2)

Stateless한 인증 정보 관리를 위해 클라이언트에서 헤더에 Authentication을 추가해 전달하자 CORS가 발생했다.

예상으로는, 이전까지는 커스텀 헤더(Authorization)가 없어서 <red>Simple Request</red> 방식으로 요청이 전달됐다.

그러나 JWT를 추가하며 요청에 Authorization 헤더가 추가되었고, 따라서 요청이 SimpleRequest에서 <red>Preflight Request</red>으로 변경되며 생긴 문제라고 추측하고 있다.

(근데 preflight는 ok됐는데?) → preflight response header에는 Access-Control-Allow-Origin이 있는데 Get요청에는 없다.

왜 없지…??  
커스텀 헤더를 삭제하면 GET 요청에서도 Access-Control-Allow-Origin이 잘 요청 된다.  
Cors 문제를 몇번을 만나봤는데(그리고 그때마다 쌩쇼를 함) 여전히 어렵다 ㅠ

<br>

## 해결

```java
http.cors(cors -> cors.configurationSource(corsConfig.corsConfigurationSource())); // CORS 설정 활성화
```

Security 단에서 Cors를 설정해주니 정상적으로 동작했다.

기존에는  원래 사용하고 있던 CorsConfig 파일을 사용하고 있었는데, 이렇게 하면 Security 필터체인이 동작하기 전에만 CosFilter가 동작하고 Security의 인증 및 권한 부여 필터와 연동되지 않는 듯…하다

(그래… 필터 페인에 CorsFilter가 따로 있는 이유가 있겠지…)

<br><br>

---

공부를 하다보면 Cors 때문에 머리를 싸매지 않을 날이 올까...

다음 게시물에선 Role을 활용한 인가처리를 다뤄볼 예정이다.