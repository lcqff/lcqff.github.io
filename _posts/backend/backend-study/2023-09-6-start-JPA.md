---
layout: post
title: "[JPA] JPA 시작하기"
author: pam
categories: Backend-Study
tags: JPA
---

## 프로젝트 생성

> 강의에서는 데이터베이스로 **H2**, 빌드 툴로 **메이븐**을 채택했지만 스프링 스터디와 프로젝트 경험으로 Gradle 사용이 익숙하고, 이미 노트북에 설치되어 있는 MySQL을 두고 H2를 새로 설치하고 싶지 않아 나는 강의와 다르게 **MySQL과 Gradle**을 사용해 프로젝트를 빌드했다.
>
> MySQL과 Gradle 사용으로 프로젝트 초기 설정이 강의와는 달랐기에, 따로 자료를 찾아보며 프로젝트를 생성했다.

### 설정 파일

![pom.xml](https://github.com/lcqff/lcqff.github.io/assets/71930280/98fbe430-a7b8-49e8-a3e0-92f9c8235302)

pom.xml은 `Marven`의 설정 파일로, Gradle을 사용할 때는 사용되지 않는다. 이때 Gradle 사용시 pom.xml의 역할을 대신하는 것이 **`build.gradle`** 이다. 

두 파일 모두 프로젝트의 이름, 버전, 그룹 id와 같은 구성과 프로젝트 빌드, 의존성 관리에 대한 정보를 담고있다. 강의 스크린샷에서 pom.xml 파일에 담긴 프로젝트 구성 정보와 **JPA 하이버네이트**, **H2 데이터베이스**에 대한 의존성 정보를 확인할 수 있다.

<br>

```java
plugins {
    id("java")
}

group = "jpa-basic"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    testImplementation(platform("org.junit:junit-bom:5.9.1"))
    testImplementation("org.junit.jupiter:junit-jupiter")

    // Hibernate
    implementation("org.hibernate:hibernate-core:5.5.6.Final")
    implementation("org.hibernate:hibernate-annotations:3.5.6-Final")

    // MySQL JDBC 드라이버
    implementation("mysql:mysql-connector-java:8.0.27")
}
 
tasks.test {
    useJUnitPlatform()
}
```

`build.gradle`은 위와 같이 작성된다. pom.xml과 동일하게 프로젝트 정보와 의존성 정보가 담겨있다. 다만, 나는 강의와 다르게 H2가 아닌 MySQL 데이터베이스를 사용하기 때문에 H2 의존성 정보 대신 MySQL 의존성 정보를 추가했다.  

<br/>

### persistence.xml

![jpa 구동방식](https://github.com/lcqff/lcqff.github.io/assets/71930280/9c25809e-6912-48f4-8a8e-ffe766d3ae4f)

`persistence.xml`은 JPA에서 필요로하는 설정을 관리하는 파일이다. 설정 파일이 `클래스패스 경로/META-INF/persistence.xml` 에 위치할 경우 별도의 설정 없이 JPA가 인식할 수 있다..

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
  xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
  <persistence-unit name="helloJPA">
    <properties>
      <!-- 필수 속성 -->
      <property name="hibernate.connection.driver_class" value="com.mysql.cj.jdbc.Driver"/>
      <property name="hibernate.connection.url" value="jdbc:mysql://localhost:3306/basicJPA"/>
      <property name="hibernate.connection.username" value="root"/>
      <property name="hibernate.connection.password" value="fostree1351"/>
      <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5InnoDBDialect"/>

      <!-- 옵션 -->
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
      <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
    </properties>
  </persistence-unit>
</persistence>
```

강의의 h2 설정을 MySQL 설정으로 바꾸어주었다. 이때 **hibernate.connection.~** 에는 연결되는 데이터베이스의 사용자 이름, 비밀번호, 주소와 같은 설정을 지정하고, **hibernate.dialect**에는 하이버네이트가 데이터베이스와 통신을 하기 위해사용하는 **방언**을 지정한다. 

![방언](https://github.com/lcqff/lcqff.github.io/assets/71930280/fbdd1a0d-ccac-4e4a-88bd-b64f40cbd00b)

방언이란 **SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능**을 의미한다.

그러므로 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르고, 사용하는 데이터베이스에 알맞는 방언을 사용하도록 주의해야한다.

<br/>

<div class="callout">
  <div>❓</div>
  <div>
    <strong>없어도 되나?</strong><br/>
    그런데 해당 `persistence.xml`이 존재하지 않아도 코드를 살행시켰을 때 정상적으로 돌아가는 걸 확인할 수 있었다. 이제 생각하면 당연한 것이, 나는 main 메소드 내에 아무것도 안넣고 실행을 눌렀기 때문이었다. 
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/7f401184-e127-417e-a024-8f669c1918cd"> 
    Persistence 클래스가 없으니 당연히 persistence.xml도 조회하지 않았을 것이고, 그렇다면 persistence.xml이 있든 없든 실행에 영향을 미치는 않을 테니 말이다. 다행히 금방 알아차렸다.
  </div>
</div>


---

## 애플리케이션 개발

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/f2959635-83ea-4940-bfe4-a96170c0849f)

JPA의 **`Persistence 클래스`**는 **`엔티티 클래스`**를 나타낸다. 그리고 **엔티티 클래스**란, 데이터베이스 테이블과 매핑되는 **데이터베이스 레코드**를 나타낸다.

```java
package helloJPA;

public class JpaMain {
  public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("helloJPA");
    EntityManager em = emf.createEntityManager();

    //code

    em.close();
    emf.close();
  }
}
```
여기서 "helloJPA"는 가져올 Persistence Unit의 이름을 의미한다. 이는 **엔티티 클래스와 데이터베이스 연결 정보, 트랜잭션 설정 및 기타 JPA 설정을 묶는 단위**로, 하나의 persistence.xml에는 여러개의 Persistence Unit이 존재할수 있기에 고유한 이름을 지녀야 한다.

<br/>

`<persistence-unit name="helloJPA">`  
Persistence.xml파일에서 설정해놓은 persistence unit 이름을 확인할 수 있다.
{: .box-note}
<br/>

**`엔티티 매니저 팩토리`**는 어플리케이션 로딩 시점에 하나만 생성되어야 한다(DB당 하나). 즉 필요할때마다 계속 **재활용된다**. 생성된 엔티티 매니저 팩토리는 일관적인 트랜젝션 범위 내에서 **엔티티 매니저**를 생성하고, 엔티티 매니저가 데이터베이스 커넥션을 얻고 쿼리를 날리면 엔티티 매니져를 종료시킨다.

그러나 **`엔티티 매니저`**는 엔티티 매니저 팩토리와 다르게 재활용될 수 없다. 회원의 요청이 들어올 때마다 새로 생성하고 요청이 끝나면 close()를 사용하여 버려야 한다. 즉 **쓰레드간 공유할 수 없다**.

<br>


```java
package helloJPA;

@Entity
public class Member {
  @Id
  private Long id;
  private String name;

	//getter, setter
}
```

- `@Entity`: 해당 클래스가 JPA를 사용함을 명시
- `@Id`: id 키가 해당 클래스의 PK임을 명시
    

```java
public class JpaMain {

  public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("helloJPA");
    EntityManager em = emf.createEntityManager();

    EntityTransaction tx = em.getTransaction();
    tx.begin();

    Member member = new Member();
    member.setId(1L);
    member.setName("memberA");
    em.persist(member);

    tx.commit();

    em.close();
    emf.close();
  }
}
```

 데이터베이스의 **일관성과 무결성을 유지**하기 위해 우리는 **트랜잭션(Transaction)**을 사용해야 한다. 

위 코드는 특정 멤버 데이터를 데이터베이스에 저장하는 코드이다(`em.persist(member);`). 이렇게 데이터베이스를 조작하면서도 데이터베이스의 일관성과 무결성 유지를 위해 트랜잭션이 사용되며, 모든 데이터베이스의 변경은 트랜젝션 내에서 수행되어야 한다.

<br>

```
Hibernate: 
    /* insert helloJPA.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

위 코드를 실행시키면 콘솔에서 **JPA가 생성한 쿼리**를 확인할 수 있다. 해당 설정은 persistence.xml에서 확인할 수 있다.

<div>
	&#60;!-- 옵션 --> <br>
  &#60;property name="hibernate.show_sql" value="true"/ &#62; <br>
  &#60;property name="hibernate.format_sql" value="true"/&#62; <br>
  &#60;property name="hibernate.use_sql_comments" value="true"/&#62;  
</div>
{: .box-note}

<br>

그러나 위 코드는 좋은 코드라고 볼 수 없다. 만일 실행 중간에 오류가 생겼을 경우 **엔티티 매니저**와 **엔티티 매니저 팩토리**를 종료시키는 코드가 실행되지 않기 때문이다. 특히 엔티티 매니저의 경우 데이터베이스 커넥션을 물고 동작하기 때문에 꼭 종료 되도록 코드를 짜야한다.

<br>

```java
package helloJPA;

public class JpaMain {

  public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("helloJPA");
    EntityManager em = emf.createEntityManager();

    EntityTransaction tx = em.getTransaction();
    tx.begin();

    try {
      Member member = new Member();
      member.setId(1L);
      member.setName("memberA");

      em.persist(member);

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }
    emf.close();
  }
}
```

### JPQL

JPA로도 조회는 가능하지만 좀 더 복잡한 조회라면 말이 다르다. 예를 들어 18세 이상인 회원만을 검색하고 싶다면? 이때 사용하는 것이 JPQL이다.

```java
try {
      List<Member> result = em.createQuery("select m from Member as m", Member.class)
          .setFirstResult(5)
          .setMaxResults(8)
          .getResultList();

      for (Member member:result) {
        System.out.printf(member.getName());
      }

      tx.commit();
    }
```

5번부터 8번까지의 항목을 리스트 형태로 가져오고, 그것을 나열하는 코드이다.  
JPQL로 짠 쿼리는 설정된 방언에 맞춰서 번역되는 번역의 역할을 하기도 한다.