---
layout: post
title: "JPA 소개"
author: 팜
categories: JPA
tags: JPA
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

## JPA 소개

JPA란 Java Persistence API의 약칭으로, 자바 진영의 **ORM** 기술 표준이다. 

<div class="callout">
  <div>💡</div>
  <div>
	<strong>ORM: Object relational mapping(객체 관계 매핑)</strong><br/>
	객체대로 설계된 객체와 데이터베이스대로 설계된 데이터베이스 사이를 매핑해주는 것이 ORM이다. ORM은 객체와 데이터베이스 사이 패러다임 불일치를 해결해준다. 
  </div>
</div>

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/77e3a397-3adc-4073-a0ad-e242d005913f)


JPA는 DB와 Java 어플리케이션 사이에 존재한다. 예전에는 개발자가 직접 JDBC API를 사용하여 DB와 통신했지만, JPA를 사용으로 개발자는 JPA에게 JDBC API를 사용을 넘겨줄 수 있게 됐다.

이렇게만 보면 JPA가 Mybatis나 JDBC 템플릿 같은 다른 자바 퍼시스턴스 프레임워크와 비슷해보일 수 있으나, JPA의 가장 큰 특징은 따로있다.

![JPA-저장](https://github.com/lcqff/lcqff.github.io/assets/71930280/35856435-9d7f-489c-bb74-01d1135b97e6)


Member 객체를 DB에 저장한다고 가정해보자. 단순히 MemberDAO가 JPA에게 Member 엔티티(객체)를 던져주는 것만으로, JPA는 **Member 엔티티를 분석하고 Insert SQL을 생성**한다. 그뿐만이 아니라 JDBC API를 사용하여 **Insert 쿼리를 실행**한다. 그리고 가장 중요하게, **패러다임 불일치를 해결**한다.

JPA를 사용하는 것으로 마치 자바 컬렉션에 객체를 저장하듯 **한 줄 만으로 데이터베이스에 객체를 저장할 수 있게 된 것**이다!

![JPA-조회](https://github.com/lcqff/lcqff.github.io/assets/71930280/17905bda-820a-4ba3-a8ad-4a2b9055dbbf)

조회하는 것 또한 간단하다. JPA에게 Id를 전달해주는 것 만으로, JPA는 우리에게 엔티티 객체를 전달해준다. 

JPA의 사용으로 판은 SQL중심적인 개발에서 객체 중심적인 개발로 이동했다. 이는 생산성의 확대를 가져왔을 뿐만 아니라 유지보수의 편리성과 성능을 증대시켰다.

### JPA와 CRUD

- 저장: `jpa.persist(member)`
- 조회: `Member member = jpa.find(memberId)`
- 수정: `member.setName(“변경할 이름”)`
- 삭제: `jpa.remove(member)`

### JPA와 유지보수

![JPA도입 이전의 방식](https://github.com/lcqff/lcqff.github.io/assets/71930280/1ea8ca58-80d7-4a94-aba2-1bd696706a80)

JPA 도입 이전, 객체에 새로운 필드가 추가되면 개발자가 필요한 SQL을 찾아다니며 해당 필드를 직접 추가해주어야 했다. 당연히 유지보수가 잘 될리가 없다.  
반면 JPA를 사용하면, 단순히 객체에 필드를 추가해주는 것으로 모든 것이 끝난다. **JPA가 모든 SQL문을 알아서 처리**하기 때문이다. 이는 개발자의 유지보수 영역을 축소시킨다.

### 신뢰할 수 있는 엔티티, 계층

```java
class MemberService {
	...
	public void process() {
		Member member = memberDAO.find(memberId);
		member.getTeam(); //자유로운 객체 그래프 탐색
		member.getOrder().getDelivery();
	}
}
```

memberDAO가 jpa를 사용한다면, **지연로딩**이라는 기술을 사용해 객체 그래프 내에 존재하는 모든 객체를 사용할 수 있게 된다. 지연로딩에 대해서는 이후에 설명한다.

### JPA의 비교

```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; //같다.
```

JPA는 동일한 트랜잭션에서 조회한 엔티티의 동일성을 보장해준다. 

### JPA의 성능 최적화

**1. 1차 캐시와 동일성(identity) 보장**

위에서 JPA는 동일 트랜젝션에서는 동일한 엔티티를 반환한다고 설명했다. 이러한 동일성 보장은 약간의 조회 성능향상을 불러오기도 한다. 

어떤 기술이 두 시스템을 연결해주는 경우(JPA가 데이터베이스와 Java어플리케이션을 연결하 듯이) 캐싱을 사용할 수 있게 된다. 이전에 조회했던 데이터를 캐시로 저장해두고, 동일한 데이터가 조회 됐을 때 캐시를 전달해주는 것이다. 

```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId); //SQL
Member member2 = jpa.find(Member.class, memberId); //캐시
```

**2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)**

어떤 기술이 두 시스템을 연결해주는 경우에 사용할 수 있는 성능 향상 기술이 하나 더 있다. 바로 **버퍼**이다.

트랜잭션을 커밋할 때까지 INSERT SQL을 모으고, JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송이 가능하다. 이를 통해 네트워크 통신 비용을 줄일 수 있다.

```java
transaction.begin(); // [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

**3. 지연 로딩(Lazy Loading)**

**지연 로딩**과 대비되는 것으로 **즉시 로딩**이 있다. 

- **지연로딩**: 객체가 실제 사용될 때 로딩한다. (연관된 객체를 함께 들고오지 않는다.)
- **즉시로딩**: 연관된 객체를 함께 들고온다. 

예를 들어 Team과 Member가 연관관계에 있다고 생각해보자. 그렇다면 Member를 가져올 때 Join을 통해 Team을 함께 들고오는 것이 성능상 유리할 것이다. 그러나 Team 객체가 쓰이는 빈도가 매우 적다면 Member만 조회하는 것이 옳다. 

그러나 연관 관계의 객체를 함께 들고오는 것이 나을지, 아니면 사용하는 객체 하나만 가져올 것인지는 쓰임마다 다를 것이고, 따라서 JPA는 지연로딩과 즉시로딩을 모두 지원한다. 

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/e26b402c-0089-431e-80c7-35540a8b1e6d)

일반적으로 JPA를 사용해 개발할 때는 지연 로딩으로 개발해두고, 이후 즉시로딩으로 변경하는 것으로 최적화한다.

JPA는 데이터베이스와 JAVA어플리케이션을 매핑하는 기술이다. 즉, JPA를 잘 알기 위해서는 JPA가 연결하고 있는 두 기술을 잘 이해하는 것이 필요하다.