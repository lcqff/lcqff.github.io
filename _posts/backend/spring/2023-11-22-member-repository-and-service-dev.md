---
layout: post
title: "레파지토리와 서비스 개발"
author: pam
categories: Backend-Study
tags: spring, jpa
---

```java
@Entity
@Getter
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    private Address address;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```

주어진 회원 도메인은 위와 같다.


---
### 회원 레파지토리 개발


```java
package jpabook.jpashop.repository;

@Repository
public class MemberRepository {
    @PersistenceContext
    private EntityManager em;

    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }

    public Member find(Long id) {
        return em.find(Member.class,id);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name",Member.class)
                .setParameter("name", name)
                .getResultList();
    }
}
```

**Repository**는 계층형 아키텍쳐에서 Persistence Layer에 속하는 개념으로, **데이터베이스나 데이터 저장소와 상호 작용**한다.  
이렇게 Repository에 데이터베이스와의 상호작용의 책임을 부여함으로써 객체 지향 프로그래밍과 데이터베이스 연동 작업을 간소화하고 추상화할 수 있다.
<br/><br/> 

지금부터 위 코드에서 사용된 개념을 하나씩 짚어서 살펴보자.

- **`@PersistenceContext`**: 컨테이너가 EntityManager를 관리하고 주입

```java
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("helloJPA");
    EntityManager em = emf.createEntityManager();

    EntityTransaction tx = em.getTransaction();
    tx.begin();

		...code

    tx.commit();

    em.close();
    emf.close();
```

기존 JPA 구현 코드와 SpringBoot에서의 JPA 코드를 비교해보자. 기존 JPA에서는 엔티티 매니저 팩토리와 엔티티 매니저를 생성 했으나 SpringBoot에서는 둘 다 생성하지 않은 것을 확인할 수 있다.

이는 **SpringBoot Starter Data JPA 라이브러리**를 사용하고 있기 때문으로, 해당 라이브러리는 Spring Data JPA 라이브러리에서 yml 설정 파일을 읽어 **엔티티 매니저 팩토리와 엔티티 매니저를 생성**하는 기능을 제공한다. 

또한 개발자가 직접 엔티티 매니저를 생성하지 않아도 `@PersistenceContext` 어노테이션을 통해 스프링에서 생성한 엔티티 매니저를 주입받는다.
<br/><br/>

- **`createQuery(String qlString, Class<T> resultClass);`**: 객체를 조회하기 위한 JPQL
    - qlString: JPQL 쿼리 문자열
    - resultClass: 쿼리 결과를 매핑할 Java 클래스

SQL과 JPQL 모두 데이터베이스에서 정보를 조회하고 수정하기 위해 사용된다. JPQL은 실행 시에 JPA 구현체에 의해 SQL로 변환되어 실제 데이터베이스에 전달되며, JPQL은 SQL에 포함되는 개념이다.
하지만 SQL은 테이블을 대상으로 쿼리를 작성하나 JPQL은 엔티티 객체를 대상으로 쿼리를 작성한다.  
따라서 JPQL은 SQL과 문법상 차이를 보인다. 위 예시의 JPQL String `select m from Member m`에서 나타나는 Member는 테이블 명이 아닌 **객체 Member**를 의미한다.


### 회원 서비스 개발

```java
package jpabook.jpashop.service;

@Service
@Transactional(readOnly = true)
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    //회원 가입
    @Transactional
    public Long join(Member member) {
        validateMember(member);
        memberRepository.save(member);
        return  member.getId();
    }

    private void validateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if (findMembers.isEmpty()) {
            return;
        }
        throw new IllegalStateException("이미 존재하는 회원입니다.");
    }

    // 회원 조회
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }
    
    public Member findMember(Long id) {
        return memberRepository.find(id);
    }
}

```

- **`@Transactional`**: 로직을 트랜젝션 내에서 수행되도록 함
- **`@Autowired`**: 자동 의존성 주입을 수행
MemberService에서 MemberRepository를 사용하기 위해 @Autowired를 사용해 레파지토리를 주입해주었다.
@Autowired를 사용하면 Spring은 자동으로 해당 타입의 빈을 찾아 필드, 메서드, 생성자를 자동으로 주입해준다. 우리는 뮈 예제에서 필드 주입을 위해 @Autowired 어노테이션을 사용했다.


#### 1. Field Injection(필드 주입)
```java
    @Autowired
    private MemberRepository memberRepository;
```
필드 주입은 사용하려는 Controller의 필드에 직접 `@Autowired`를 부여하는 방법이다.  
그러나 Field Injection 사용시 final 제어자를 사용이 불가능하며, 객체의 변경 가능성을 열어두어 Service의 불변성을 보장하지 못한다.
또한 테스트 코드 작성시 주입받는 객체의 생성이 어려우며, 따라서 단위 테스트가 불가능하다. 따라서 Field Injection은 지양해 사용하여야 한다.

#### 2. Setter Injection(수정자 주입)

```java
    private MemberRepository memberRepository;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```
위와 같이 Setter Injection을 사용하면 테스트 코드를 작성할 때 개발자가 직접 주입이 가능하다는 장점이 있다.  
그러나 실제로 애플리케이션 동작 중에 객체를 변경하게 될 일은 거의 없으므로, 잘 쓰이지 않는 주입법이다.

#### 3. Constructor Injection(생성자 주입)
```java
    private final MemberRepository memberRepository;

    //@Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```
가장 권장되는 Injection 방식이다.  
생성자 주입은 Service를 생성할 때 객체의 주입을 완료하기 때문에 중간에 객체의 변경이 불가능하며, 의존중인 객체들 중 하나라도 주어지지 않으면 객체 생성이 불가능하다.
또한 final 제어자를 사용해 불변성을 보장할 수 있다.
`@Autowired `어노테이션을 작성하지 않아도 생성자를 작성할 경우 자동으로 Injection을 하는 기능을 Spring이 제공하나. 생성자 작성 대신 Lombok에서 제공하는 `@RequiredArgsConsturctor` 어노테이션을 사용할 수도 있다.

- **`@RequiredArgsConsturctor`**: 생성자를 작성하지 않아도 final 제어자가 있는 필드에 대해서 생성자를 생성해준다.



### 회원 기능 테스트

```java
package jpabook.jpashop.repository;

@ExtendWith(SpringExtension.class)
@SpringBootTest
class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    public void testMember () {
        Member member = new Member();
        member.setUserName("memberA");
        
        Long saveid = memberRepository.save(member);
        Member findMember = memberRepository.find(saveid);

        assertEquals(findMember.getId(), member.getId());
        assertEquals(findMember.getUserName(), member.getUserName());
    }
}
```

- **`@ExtendWith(SpringExtension.class)`**: JUnit에게 스프링 관련 테스트를 함을 알리기 위한 어노테이션
- **`@SpringBootTest`**: 스프링 부트로 테스트하기 위한 어노테이션

<br/>

**No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call** 
{: .box-error}

위 테스트 코드를 실행하면 위와 같은 에러가 발생한다. 에러를 보면 알 수 있듯이 이는 트랜잭션이 존재하지 않기에 발생하는 문제이다.

엔티티 매니저를 통한 모든 변경은 트랜잭션 안에서 이루어져야 한다. 

```java
 		@Test
        @Transactional
		@Rollback(value = false)
    public void testMember () {
        Member member = new Member();
        member.setUserName("memberA");
        
        Long saveid = memberRepository.save(member);
        Member findMember = memberRepository.find(saveid);
        memberRepository.find(saveid);

        assertEquals(findMember.getId(), member.getId());
        assertEquals(findMember.getUserName(), member.getUserName());

    }
```

- **`@Transactional`**: 스프링의 선언적 트랜잭션 설정 어노테이션
    - 테스트에 존재할 시 테스트 종료 후 DB를 롤백한다.
- **`@Rollback(value = false)`**: Transactional의 DB 롤백 설정을 False로 한다.

`@Rollback(value = false)` 어노테이션 없이 `@Transactional`만 사용할 경우 Insert 쿼리가 실행되지 않음을 확인할 수 있다.
쿼리는 영속성 컨텍스트 내에 있는 멤버 객체가 flush가 될 때 실행되는데, `@Transactional` 어노테이션이 트랜잭션의 커밋 이전에 롤백을 하기 때문이다.
실제로 `@Rollback(value = false)`을 사용하지 않을 경우 테스트 코드를 실행해도 데이터베이스에 저장되지 않음을 확인할 수 있다.