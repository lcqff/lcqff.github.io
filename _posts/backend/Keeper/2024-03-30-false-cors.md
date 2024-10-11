---
layout: post
title: "가짜 Cors 해결(false cors)"
author: pam
categories: Keeper
tags: Keeper bug
sidebar:
---
## 문제

![요청](https://github.com/lcqff/lcqff.github.io/assets/71930280/02c41f91-b918-4f20-8017-9e220c28563b)
<br>
위와 같은 버그 해결 요청이 들어왔다. 모두 Cors 에러가 발생하며 Allow-Cors를 사용할 시 정상동작하기에 Cors 문제라고 판단되었다.

### 에러 재현

![애러재현1](https://github.com/lcqff/lcqff.github.io/assets/71930280/2ba5def4-e141-4577-9b97-4294c77063b3)

로그인 페이지 리다이렉션 요청 씹힘

- 로그아웃 상태로 홈페이지 접근시 CORS 오류가 발생하며 로그인 화면이 뜨지 않음

![에러재현2](https://github.com/lcqff/lcqff.github.io/assets/71930280/3f1d64fb-4250-490b-a792-c8c7e7e42032)

메일 인증 요청시 CORS 오류 발생하며 인증 메일이 전송되지 않음
<div class="captionedImg">
  <div class="imageRow">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/7d183764-984b-4529-b53d-9701eab9c803">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/bf6a33ec-a882-42c9-b938-91227fe29597">
  </div>
  <p>토스트 정상 동작</p>
</div>

<div class="captionedImg">
  <div class="imageRow">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/c46d4300-fb23-49d4-95cf-602ad5d30c62">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/9cc5a561-3084-4dca-b73c-a36468a6b02a">
  </div>
  <p>토스트 Cors 오류</p>
</div>

댓글 작성 없이 파일 다운로드 시 떠야하는 토스트 메세지 씹힘

문제를 해결하기 위해 우선 Cors가 무엇인지에 대해 알아보자.

## Cors?

{: .box-error}
> Access to XMLHttpRequest at 'https://api.keeper.or.kr/posts/notices?categoryId=102' from origin '[https://keeper.or.kr](https://keeper.or.kr/)' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
> 

웹 브라우저는 기본적으로 동일 출처 정책(Same-Origin Policy)을 따르기 때문에 동일한 출처에서 온 스크립트 혹은 요청이 아니면 리소스애 대한 접근을 차단한다. → **Cors 오류**

<div class="callout">
  <div>💡</div>
  <div> 
    <strong>Origin</strong><br/>
    특정 리소스에 접근하는 웹 페이지의 출처로, 프로토콜(protocol), 호스트(host), 포트(port)로 정의된다.<br/>
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/0398ae78-bcba-44d3-a9a0-65eef70bbc61" />
  </div>
</div>

<div class="callout">
  <div>💡</div>
  <div> 
    <strong>CORS(Cross-Origin Resource Sharing) 란?</strong><br/>
    웹 브라우저에서 발생하는 보안 정책 중 하나로, 웹 브라우저가 다른 도메인(Origin)의 리소스에 접근할 수 있도록 하는 설정<br/>
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/2e585ff7-9d00-4c33-818c-d22146ff2b80" />
  </div>
</div>


 일반적으로 HTTP header에 추가적인 정보(**Access-Control-Allow-Origin**)를 추가하여 서버가 브라우저에게 자기 자신뿐만 아니라 다른 origin 에서 요청한 정보도 허용할 수 있도록 알려준다.

cors 설정을 통해 필요한 경우에만 다른 출처의 리소스에 접근할 수 있으며, 보안을 강화한다.

## CORS Preflight

### preflight

![how preflight works](https://github.com/lcqff/lcqff.github.io/assets/71930280/d30113d7-9471-4e12-906f-436f2ad16115)

브라우저는 요청이 접근 가능한 Origin에서 왔다는 걸 판단하기 위해 **CORS Preflight** 요청을 보낸다.

해당 **Cors preflight값 요청 결과**에 따라 브라우저는 본 요청을 서버로 보낼지 혹은 차단할지(Cors error) 결정한다.

- preflight 요청: 본 요청이 **[Simple Request](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#%EB%8B%A8%EC%88%9C_%EC%9A%94%EC%B2%ADsimple_requests)**가 아닌 경우 발생
- 실제 요청: JavaScript 상에서 XMLHttpRequest나 Fetch API 등을 사용한 요청

### OPTIONS

![options](https://github.com/lcqff/lcqff.github.io/assets/71930280/03a97625-bd93-4198-b63c-327082224c76)

이때 Cors preflight 요청으로 **OPTIONS**가 사용된다.

#### options 요청&응답 예시

```yaml
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.example
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, 
Content-Type
```

```yaml
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
**Access-Control-Allow-Origin: https://foo.example**
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
```

### cURL 테스트

Cors 에러는 브라우저를 통해 발생하는 에러로, 로컬 서버로는 동일한 에러를 재현하기 어렵다.  

그러나 방법 중 하나로, 로컬에서 **cURL 테스트**를 통해 Cors 에러를 확인할 수 있다. 

```yaml
  **curl -v -X OPTIONS https://api.keeper.or.kr/sign-up/email-auth -H "Origin: https://keeper.or.kr" -H "Access-Control-Request-Method: POST";
  //......
  > OPTIONS /sign-up/email-auth HTTP/1.1**
  > Host: api.keeper.or.kr
  > User-Agent: curl/8.1.2
  > Accept: */*
  **> Origin: https://keeper.or.kr**
  > Access-Control-Request-Method: POST

  //응답
  **< HTTP/1.1 200**
  < Server: nginx
  < Date: Tue, 26 Mar 2024 07:50:11 GMT
  < Content-Length: 0
  < Connection: keep-alive
  < Vary: Origin
  < Vary: Access-Control-Request-Method
  < Vary: Access-Control-Request-Headers
  < Access-Control-Allow-Methods: POST
  < Access-Control-Allow-Credentials: true
  < X-Content-Type-Options: nosniff
  < X-XSS-Protection: 0
  < Cache-Control: no-cache, no-store, max-age=0, must-revalidate
  < Pragma: no-cache
  < Expires: 0
  < X-Frame-Options: DENY
  **< Access-Control-Allow-Origin: https://keeper.or.kr**
  <
  * Connection #0 to host api.keeper.or.kr left intact
```

> 200이 리턴되었다면 Cors에러가 발생하지 않았다는 뜻이다.


![preflight](https://github.com/lcqff/lcqff.github.io/assets/71930280/77403046-f652-4cdd-8aaf-496122cdf5da)

Cors는 웹 브라우저에서 시행되는 정책이긴 하나 실제로 Cors에 위반했는지를 판단하는 건 브라우저가 아닌 서버이다. 브라우저는 서버로 Cors 여부를 확인하는 요청인 Cors preflight을 보내고, 요청에 대한 서버의 응답값을 확인한 후 본 요청을 보내거나 차단한다.

따라서 브라우저 없이도 Cors preflight를 서버에 전송할수만 있다면 Cors 여부를 판단할 수 있다.

이때 Cors preflight를 모방하기 위해선 헤더에 반드시 Origin을 작성해야한다. 

### 😯 OPTIONS 명령 없이도 Cors가 발생했는데요?

![에러](https://github.com/lcqff/lcqff.github.io/assets/71930280/c87b8180-911a-4605-b7d0-42c7cd002e40)

위에서 Cors 여부를 판단하기 위해선 본 요청 이전에 예비 요청인 Cors preflight가 필요하다 설명했다.

그러나 사진과 같이 Options 명령 없이도 Cors에러가 발생할 때가 있다. (빨간줄 모두 Get 명령임)

**Cors preflight가 발생하지 않음에도 Cors 에러를 발생시킬수 있는 경우는 아래 4가지 정도가 있다.**

1. **Simple request인 경우**
2. **Credentialed Request인 경우** 
3. **Preflight가 캐싱된 경우**
4. **가짜 Cors의 경우**

## Cors preflight가 발생하지 않는 요청

### 1. Simle Request

본 요청이 **[Simple Request](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#%EB%8B%A8%EC%88%9C_%EC%9A%94%EC%B2%ADsimple_requests)**인 경우 preflight(Option 명령)은 발생하지 않는다. 
브라우저는 Preflight 응답이 아닌 **본 요청의 응답 헤더 Access-Control-Allow-Origin 값을 통해 Cors 정책 위반 여부를 확인**한다.

Simple Request가 되기 위해서는 아래의 모든 조건을 만족해야한다.

> 1. 다음 Method 중 하나여야 한다.
> 
> - [GET](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)
> - [HEAD](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)
> - [POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)
> 
> 1. [Connection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection), [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent), or [the other headers defined in the Fetch spec as a *forbidden header name*](https://fetch.spec.whatwg.org/#forbidden-header-name)),와 같이 자동 설정되는 헤더를 제외하고 [수동 설정할 수 있는 헤더](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)는 다음으로 제한된다:
> - [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)
> - [Accept-Language](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language)
> - [Content-Language](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language)
> - [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)(아래 내용 참고)
> - [Range](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range) (only with a [simple range header value](https://fetch.spec.whatwg.org/#simple-range-header-value); e.g., `bytes=256-` or `bytes=127-255`)
> 
> 1. [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header의 값은 다음으로 제한된다.
> - `application/x-www-form-urlencoded`
> - `multipart/form-data`
> - `text/plain`

위와 같은 조건을 보면 알겠다 싶이 거의 대부분의 요청은 Simple Request가 아니다.<br>

![simple request](https://github.com/lcqff/lcqff.github.io/assets/71930280/ed7ca2d3-f4f2-46c6-9902-5118b9647cc2)

```yaml
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example

HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
**Access-Control-Allow-Origin: https://foo.example**
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```

### 2. Credentialed Request

보안상 이유로 출처가 다른 경우 사용자 인증 정보를 담은 요청을 **Cors를 통해** 브라우저상에서 제한하고 있다. 

사용자 인증 정보를 다른 출처의 서버로 전송하기 위해선 Credentialed Request를 사용해야한다. 

> 서버와 클라이언트(백엔드 프론트) 양측의 Credentialed Request 설정이 필요하다.

![Credentialed Request](https://github.com/lcqff/lcqff.github.io/assets/71930280/d07f50cf-9472-42ab-81cf-689c894c0c67)

```yaml
GET /resources/credentialed-content/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Referer: https://foo.example/examples/credential.html
Origin: https://foo.example
**Cookie: pageAccess=2 //사용자 인증 정보를 담은 요청**

HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:34:52 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
**Access-Control-Allow-Credentials: true**
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: pageAccess=3; expires=Wed, 31-Dec-2008 01:34:53 GMT
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 106
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain

[text/plain payload]
```

### 3. Preflight 캐싱

모든 요청에 대해 예비 요청이 함께 발생시키는 것은 서버에 부담을 주고 실제 요청까지 걸리는 시간을 잡아먹는다. 

따라서 **Preflight의 캐싱**을 통해 서비스의 성능 향상을 향상시킬 수 있다.

preflight 응답에 다음 헤더를 추가하는 것으로 preflight를 캐싱할 수 있다.

```java
Access-Control-Max-Age: 3600
```

```java
  public void addCorsMappings(CorsRegistry registry) {
    registry.addMapping("/**")
        .allowedOrigins("https://keeper.or.kr", "https://localhost:3000")
        .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
        .allowedHeaders("headers")
        .**maxAge(3000);**
  }

```

캐싱된 시간동안은 새로운 **Options 요청이 발생하지 않으며**, 캐싱된 Options 응답값을 사용한다.

### 4. 가짜 Cors

Options 명령이 발생하지 않는 3가지 경우에 대해 습득했다. 

해당 내용을 기반으로 발견한 오류에 대해 다시 살펴보자.

![에러](https://github.com/lcqff/lcqff.github.io/assets/71930280/c87b8180-911a-4605-b7d0-42c7cd002e40)

<br>
Options 명령이 발생하지 않았고, Cors error가 발생했다. 이때 3가지 경우를 살펴볼 수 있다.

1. `Simple request`인가?
    1. GET, HEAD, POST 메서드 중 하나인가?: true
    2. 적절한 요청 헤더를 가지고 있는가? : **False**
        
        ![에러 헤더](https://github.com/lcqff/lcqff.github.io/assets/71930280/8da12028-b46b-4766-a90c-67b1a7da3263)

2. `Credentialed Request`인가?
    
    응답 헤더에 `Access-Control-Allow-Credentials`가 true로 설정되어있기는 하나, 요청 헤더에 달리 사용자 인증 정보가 담겨있지 않다.
    무엇보다 일단 프론트 쪽에서 Credentialed Request 설정이 되어있지 않아 Credentialed Request 요청이 아니다.
    
    ```jsx
    const useGetPostListQuery = ({ categoryId, searchType, search, page, size }: BoardSearch) => {
      const fetcher = () =>
        axios.get('/posts', { params: { categoryId, searchType, search, page, size } }).then(({ data }) => data);
    
      return useQuery<BoardPosts>(['posts', categoryId, searchType, search, page, size], fetcher, {
        keepPreviousData: true,
      });
    };
    ```
    
3. Preflight가 캐싱 돼 있는가?
    
    ```java
      public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedOrigins("https://keeper.or.kr", "https://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
            .allowedHeaders("headers")
            .**maxAge(3000);**
      }
    
    ```
    
    다음과 같이 preflight 캐싱 정보가 포함되어 있긴 하나 최초 요청에서도(혹은 설정된 `Access-Control-Max-Age` 이후에도) Preflight를 확인하지 못했다.
    

결과적으로 해당 에러는 위의 모든 경우와 일치하지 않았다. 그럼 뭐란 걸까?

#### 세상에는 **가짜 Cors** 라는 것이 있다

일단 Options 명령이 발생하지 않았으니 cUrl 테스트를 통해 Preflight 요청을 보내보자.

```yaml
  **curl -v -X OPTIONS https://api.keeper.or.kr/sign-up/email-auth -H "Origin: https://keeper.or.kr" -H "Access-Control-Request-Method: POST";
  //......
  > OPTIONS /sign-up/email-auth HTTP/1.1**
  > Host: api.keeper.or.kr
  > User-Agent: curl/8.1.2
  > Accept: */*
  **> Origin: https://keeper.or.kr**
  > Access-Control-Request-Method: POST

  //응답
  **< HTTP/1.1 200**
  < Server: nginx
  < Date: Tue, 26 Mar 2024 07:50:11 GMT
  < Content-Length: 0
  < Connection: keep-alive
  < Vary: Origin
  < Vary: Access-Control-Request-Method
  < Vary: Access-Control-Request-Headers
  < Access-Control-Allow-Methods: POST
  < Access-Control-Allow-Credentials: true
  < X-Content-Type-Options: nosniff
  < X-XSS-Protection: 0
  < Cache-Control: no-cache, no-store, max-age=0, must-revalidate
  < Pragma: no-cache
  < Expires: 0
  < X-Frame-Options: DENY
  **< Access-Control-Allow-Origin: https://keeper.or.kr**
  <
  * Connection #0 to host api.keeper.or.kr left intact
```

🤔 성공적으로 200 ok를 반환했다.

이 뜻은 무엇이냐 하면, 브라우저는 Cors 에러가 발생했다고 인지했으나 실제로는 Cors가 발생하지 않은 것이다. (아님 내가 뭘 잘못하고 있거나)

어떻게 이런 일이 가능한 걸까?

인터넷을 뒤져보니 위와 같은 경우에 대한 글을 몇가지 찾을 수 있었다:

[What is CORS?](https://dev.to/jpomykala/what-is-cors-11kf)

[Misleading CORS Errors – Dev Notes](https://davidtruxall.com/misleading-cors-errors/)

[CORS: Why do I get successful preflight OPTIONS, but still get CORS error with post?](https://stackoverflow.com/questions/71948888/cors-why-do-i-get-successful-preflight-options-but-still-get-cors-error-with-p)

[CORS, Preflight, 인증 처리 관련 삽질](https://www.popit.kr/cors-preflight-인증-처리-관련-삽질/)

<br>
false cors, misleading cors 등 서로 다른 용어를 사용하는 거 보니 통일된 용어가 없어서 서치가 힘들었던 모양…

Cors가 처리되기 전에 중간 레이어에서 오류가 발생하는 경우, Cors 처리 전에 미들웨어가 종료되는 경우 등등 원인은 다양했으나 결국 모두 어떤 **에러 전에 헤더에 Cors가 추가되지 않았기 때문에** 브라우저가 이를 Cors로 판단해 발생하는 문제다. 

keeper의 코드 상 Cors 코드는 정상적으로 작성되어 있기에 처음에는 필터 체인의 순서 문제라 생각했다. 그러나 이후 필터 체인에서도 인증 처리 이전에 Cors가 처리됨을 확인했다.

문제 원인은 3가지로 예상했다.

{: .box-yello}
1. 인프라 설정 문제
2. 프론트에서  origin 헤더 담아 보내주기
3. 프론트에서의 credential request 설정

그러나 Cors는 대부분 프론트에서 뭘 만진다고 해결되는 문제가 아니기 때문에 2번은 아닐거라 예상했고, 3번 또한 해당 요청이 credential request가 맞는데 내가 잘못 판단한 경우를 고려한 해결방법이었다.

인프라 설정을 직접 확인하고 싶었으나 해당 프로젝트는 우리 동아리에서 다른 동아리 홈페이지를 관리하는 프로젝트였기 때문에 보안 문제상 인프라 접근이 제한되어 확인이 어려웠다…🥲 

## 문제 해결

위와 같은 결과 보고를 하고 얼마 뒤, Cors 오류가 해결되었다는 이야기를 전해들었다. **NginX 설정**이 원인이었다!

```jsx
location / {
    proxy_pass http://spring;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_hide_header Access-Control-Allow-Origin;
    add_header 'Access-Control-Allow-Origin' 'https://keeper.or.kr' **always**;
    add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization' **always**;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, DELETE, PATCH, OPTIONS' **always**;
    # add_header 'Access-Control-Allow-Origin' 'https://api.keeper.or.kr';
    # add_header 'Access-Control-Allow-Origin' '*';
    # add_header 'Access-Control-Allow-Origin' 'https://localhost:3000';
    limit_req zone=keeper_req burst=5 nodelay;
    limit_conn keeper_conn 10;
    limit_req_log_level error;
}
```

기존 NginX 코드에서 *always가 없었던 부분*이 문제였다. 

`always`가 없으면 400, 500번대 에러가 발생할시 Header에 Cors 설정값이 추가되지 않는다. always를 추가함으로써 응답 코드와 상관없이 항상 header에 cors 설정이 들어가게 된다.

Security 필터에서 발생한 401에러로 인해 Cors 설정이 되지 않아 Cors 에러가 발생한 듯하다.

---

<br>

> 후기: 정말로 이제는 인프라 공부를 하자….

그래도 이번 기회로 Cors에 대해 빡세게 공부한 거 같다😄