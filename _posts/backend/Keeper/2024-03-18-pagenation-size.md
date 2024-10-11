---
layout: post
title: "페이지네이션 크기(Size) 제한"
author: pam
categories: Keeper
tags: Keeper bug
sidebar:
---
![요청](https://github.com/lcqff/lcqff.github.io/assets/71930280/e09c7f90-d9ee-4d6b-8cb7-18385e1d6fd4)

## 문제

```java
  @GetMapping("/members/{memberId}")
  public ResponseEntity<Page<MemberPostResponse>> getMemberPosts(
      @PathVariable long memberId,
      @RequestParam(defaultValue = "0") @PositiveOrZero int page,
      @RequestParam(defaultValue = "10") @PositiveOrZero int size
  ) {
    Page<MemberPostResponse> responses = postService.getMemberPosts(memberId,
        PageRequest.of(page, size, Sort.by(DESC, "registerTime")));
    return ResponseEntity.ok(responses);
  }
```

위와 같이 page가 매핑 코드에서 size값의 크기를 제한해달라는 요청이 들어왔다. 

위 코드에서 page는 page 번호, size는 한 페이지에서 불러올 데이터의 개수이다.

만일 size의 크기를 제한하지 않는다면, 클라이언트에서 size에 매우 큰 값을 넣어 요청할 경우 **응답 시간이 지연되고 서버 부담이 커지기** 때문에 요청 가능한 최대 size를 설정하는 것이 좋다.

## 해결

[max-page-size](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#spring.data.web.pageable.max-page-size)를 통해 요청가능한 page 최대 size를 yml 파일 내에 설정할 수 있다.

| Name | Description | default value |
| --- | --- | --- |
| spring.data.web.pageable.max-page-size | Maximum page size to be accepted. | 2000 |

```yaml
  data:
    web:
      pageable:
        max-page-size: 100 
        default-page-size: 10
```

기본적인 Page 크기를 10, 최대 Page 크기를 100으로 설정했다.

```java
  @Test
  @DisplayName("페이지네이션 최대 사이즈를 넘은 요청은 최대 사이즈로 요청된다.")
  public void 페이지네이션_최대_사이즈를_넘은_요청은_최대_사이즈로_요청된다() throws Exception {
    //given
    final int MAX_PAGE_SIZE = 100;
    List<Post> posts = IntStream.rangeClosed(1, 101)
        .mapToObj(i -> Post.builder()
            .title("title")
            .content("ABCD")
            .member(memberTestHelper.generate())
            .category(category)
            .visitCount(10)
            .ipAddress(WebUtil.getUserIP())
            .allowComment(true)
            .isNotice(false)
            .isSecret(false)
            .isTemp(false)
            .password("asdf")
            .thumbnail(thumbnailTestHelper.generateThumbnail())
            .build())
        .toList();

    postRepository.saveAll(posts);

    Pageable pageable = PageRequest.of(0, 101);
    
    //when
    Page<PostResponse> foundPosts = postService.getPosts(category.getId(), "title+content", "bc", pageable);
    
    //then
    assertThat(foundPosts).hasSize(MAX_PAGE_SIZE);
  }
```

설정이 잘 적용되었는지 확인하기 위해 service단에 테스트 코드를 작성했다. 정상적으로 작동한다면 테스트 코드에서 size로 101을 입력하여도 결과 size는 100으로 반환되어야 한다.

## 에러 발생

{: .box-error}
java.lang.AssertionError: <br>
Expected size: 100 but was: 101 in

![페페](https://github.com/lcqff/lcqff.github.io/assets/71930280/ff77dce7-9d90-40c1-87a4-1952b3356ad2)

적용이 안된다… 단순히 yml에 설정을 추가하는 것이니 안될 거라곤 전혀 예상하지 못했다.

설정 자체가 틀렸거나 수정된 yml 파일이 적용이 안된 경우를 고려하여 직접 `WebMvcConfiguration`에 로직을 추가해보기도 했으나(아래 코드) 이 또한 적용되지 않았다.

```java
@Override
public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
  PageableHandlerMethodArgumentResolver pageableHandlerMethodArgumentResolver = new PageableHandlerMethodArgumentResolver();
  pageableHandlerMethodArgumentResolver.setMaxPageSize(100);
  resolvers.add(pageableHandlerMethodArgumentResolver);
  resolvers.add(loginMemberArgumentResolver);
}
```

아래로는 문제의 원인을 해결하기 위해 어떤 뻘짓을 했는지 서술할 것이다.

### 구조

![구조](https://github.com/lcqff/lcqff.github.io/assets/71930280/4cf0e541-cf43-43e8-b4ce-517484c14ada)

yml 파일의 max-page-size 설정을 타고 내려가보면 위와 같은 클래스들을 밟을 수 있다.

```java
	public void setMaxPageSize(int maxPageSize) {
		this.maxPageSize = maxPageSize;
	}
```

**SpringDataWebProperties**에서 yml 설정으로 저장된 maxPageSize가 적용되고, **PageableHandlerMethodArgumentResolverCustomizer**는 getMaxPageSize 메서드를 통해 저장된 maxPageSize를 가져온다.

```java
@Bean
@ConditionalOnMissingBean
public PageableHandlerMethodArgumentResolverCustomizer pageableCustomizer() {
	return (resolver) -> {
		Pageable pageable = this.properties.getPageable();
		resolver.setPageParameterName(pageable.getPageParameter());
		resolver.setSizeParameterName(pageable.getSizeParameter());
		resolver.setOneIndexedParameters(pageable.isOneIndexedParameters());
		resolver.setPrefix(pageable.getPrefix());
		resolver.setQualifierDelimiter(pageable.getQualifierDelimiter());
		resolver.setFallbackPageable(PageRequest.of(0, pageable.getDefaultPageSize()));
		**resolver.setMaxPageSize(pageable.getMaxPageSize());**
	};
}
```

**PageableHandlerMethodArgumentResolverCustomizer**는 **PageableHandlerMethodArgumentResolver**의 설정을 커스텀하는 인터페이스로, `resolver.setMaxPageSize(pageable.getMaxPageSize());`**을 통해 PageableHandlerMethodArgumentResolverSupport**의 `setMaxPageSize` 메서드가 실행된다.

```java
public void setMaxPageSize(int maxPageSize) {
	this.maxPageSize = maxPageSize;
}
```

### 디버깅

우선 정상적으로 설정값이 변경되었나 확인하기 위해 `PageableHandlerMethodArgumentResolverSupport`의 `setMaxPageSize` 메서드에 브레이크를 걸었다.

![브레이크](https://github.com/lcqff/lcqff.github.io/assets/71930280/7bbf2ed9-29fa-4b7e-82e3-0e6ec3fd8eb9)

![브레이크 후](https://github.com/lcqff/lcqff.github.io/assets/71930280/96daa8be-d708-453e-8d11-b2559dd52a7e)

2000(디폴트 값)으로 설정돼있던 maxPageSize가 정상적으로 100으로 변경된 것을 확인할 수 있었다.

따라서 `PageableHandlerMethodArgumentResolverSupport`를 상속받고 있는`PageableHandlerMethodArgumentResolver` 에서 초기화가 발생해 설정값이 덮어씌워진 것으로 예상했다.

![브레이크 안걸림](https://github.com/lcqff/lcqff.github.io/assets/71930280/97c5aa3e-0700-402e-9bf0-6c6539290c98)

따라서 `PageableHandlerMethodArgumentResolver`의 pageSize를 설정하는 메서드에 브레이크를 걸었으나 **브레이크가 걸리지 않고 그대로 테스트가 종료**되었다.

![흠터레스팅](https://github.com/lcqff/lcqff.github.io/assets/71930280/5b0155f8-6de7-44bf-8f44-1dd3673c3f9e)

Pageable의 설정은 변경되었으나 어떠한 이유로 **PageableHandlerMethodArgumentResolver**의 메서드가 실행되지 않아 설정 값으로 Pageable이 설정되지 않은 것으로 판단했다.

### 원인

![document](https://github.com/lcqff/lcqff.github.io/assets/71930280/029dc0a7-b3eb-4e1b-ad25-5ae2aeb99f1f)

> Extracts paging information from web requests and thus allows injecting [Pageable](https://docs.spring.io/spring-data/data-commons/docs/1.7.0.M1/api/org/springframework/data/domain/Pageable.html) instances into **controller methods.**


😄… **PageableHandlerMethodArgumentResolver는** 페이징 정보를 추출하여 Pageable 인스턴스를 컨트롤러 메서드의 파라미터에 주입하는 클래스였다.

무슨 뜻이냐 하면 설정 자체는 정상적으로 적용되었으나 **Service 계층에서 실행된 test 코드는 Controller 코드를 거치지 않기 때문에 Pageable 값이 설정되지 않는다**는 뜻이다.

나는 바보…

## 최종 해결

참고로 **PageableHandlerMethodArgumentResolver**는 Controller의 Pageable 매개변수를 주입하는 코드이므로, Controller 매개변수에 size page가 아닌 Pageable을 사용해야한다.

```java
  @GetMapping
  public ResponseEntity<Page<PostResponse>> getPosts(
      @RequestParam long categoryId,
      @RequestParam(required = false) String searchType,
      @RequestParam(required = false) String search,
      @RequestParam(defaultValue = "0") @PositiveOrZero int page,
      @RequestParam(defaultValue = "10") @PositiveOrZero int size
  ) {
    return ResponseEntity.ok(postService.getPosts(categoryId, searchType, search, PageRequest.of(page, size)));
  }
```

따라서 이전 코드는 위와 같았으나

```java
  @GetMapping
  public ResponseEntity<Page<PostResponse>> getPosts(
      @RequestParam long categoryId,
      @RequestParam(required = false) String searchType,
      @RequestParam(required = false) String search,
      @PageableDefault Pageable pageable
  ) {
    return ResponseEntity.ok(postService.getPosts(categoryId, searchType, search, pageable));
  }
```

다음과 같이 수정해주었다.

```java
  @DisplayName("최대 제한값을 초과한 size가 요청될 시 size는 최대 제한값으로 설정된다.")
    public void 최대_제한값을_초과한_size가_요청될시_size는_최대_제한값으로_설정된다() throws Exception {
      final int max_size = 100;
      params.add("categoryId", String.valueOf(category.getId()));
      params.add("searchType", null);
      params.add("search", null);
      params.add("page", "0");
      params.add("size", "1000");

      callGetPostsApi(memberToken, params)
          .andExpect(status().isOk()).andReturn();

      ArgumentCaptor<Pageable> pageableCaptor = ArgumentCaptor.forClass(Pageable.class);
      verify(postService).getPosts(any(Long.class), any(), any(), pageableCaptor.capture());
      Pageable capturedPageable = pageableCaptor.getValue();

      assertEquals(max_size, capturedPageable.getPageSize());
    }
```

테스트 코드도 Service단이 아닌 Controller 단으로 옮겨주었다.

아쉬운 점이 있다면 json 필드 값 확인으로 바로 size를 확인하고 싶었는데  응답 json을 page로 매핑하는 방법을 모르겠어서 전달된 pageable 인자를 캡쳐하는 식으로 테스트 했다… 

---

<br>
> 배운 점: Spring 내부 인터페이스와 클래스에 대한 이해도를 높이자~
>