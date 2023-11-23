---
layout: post
title: "객체와 데이터베이스간 불일치"
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

> JPA를 사용하는 큰 이유 중 하나는 객체와 데이터베이스 간의 매핑(ORM)에 있다. <br/>
> 해당 게시글에서는 객체와 데이터베이스 간의 차이와, 그로 인해 발생하는 문제점에 대해 다룬다.

## SQL 중심적인 개발의 문제점

객체를 저장하는 수단은 RDB, NoSQL, File 등 다양하게 존재하나 현실적으로 쓰이는 대안은 **RDB(관계형 데이터베이스)**이다. (NoSQL은 보통 RDB의 보조적인 수단으로 쓰인다.)
 그렇다면 관계형 데이터베이스에 객체를 저장하기 위해선 어떻게 해야하는가?


  
![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/28a96357-85dd-4d16-8b72-306b8f956f80)

데이터베이스에 객체를 저장하기 위해선 객체를 **SQL로 변환**하는 선수과정이 필요하다. 회원 정보를 DB에 저장할 때는 insert문을 사용해 SQL로 변환시켜 RDB에 전달하고, 반대로 조회할 때 또한 select문을 작성해 DB에 저장된 데이터를 객체로 바꾼다.
이런 **SQL 매핑**은, 개발자에게 불가피한 과정이다.

그렇다면 SQL을 배우기 전 객체와 관계형 데이터의 차이를 먼저 알아보자.

## 객체와 관계형 데이터베이스의 차이

객체와 관계형 데이터베이스 사이에는 크게 **상속, 연관관계, 데이터 타입, 데이터 식별 방법**의 4가지 차이점이 있다.

## 상속

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/54d4b282-906d-4584-9d1c-9350be628824)

데이터베이스엔 객체에서 흔히 말하는 상속관계가 존재하지 않는다. 그렇다면 객체의 상속관계를 데이터베이스에 저장하기 위해선 어떻게 해야하는가?

### 슈퍼타입 서브타입
객체의 상속관계를 데이터베이스에 저장하기 위해 데이터베이스 테이블 설계 기법 중 하나인 **슈퍼타입 서브타입 관계**를 사용할 수 있다. 슈퍼타입 서브타입 관계란 부모와 자식에 해당하는 객체들을 테이블로 만든 후 SQL을 사용해 상속관계를 흉내내는 기법이다.


그러나 위 방식을 사용하면 곤란한 부분이 있다. 예를 들어 Album 객체를 데이터베이스에 저장하는 과정을 생각해보자.  
<p style="font-size: 1rem;">
 1. 먼저 Album 객체를 분해해야한다.<br/>
 2. 분해된 객체의 데이터(id, 이름, 가격)를 Insert문을 사용해 ITEM 테이블에 저장한다.<br/> 
 3. 2와 동일한 방식으로 데이터(id, 작곡가)를 ALBUM 테이블에 저장한다.<br/>
</p>
{: .box-yello}


Album을 조회하는 경우는 더 까다롭다.
<p style="font-size: 1rem;">
1. ITEM 테이블과 ALBUM테이블을 Join하기 위한 SQL을 작성한다.<br/>
2. 객체를 생성한다.<br/>
3. 조인된 테이블의 데이터를 객체에 다 집어넣는다.<br/>
4. … 생략<br/>
</p>
{: .box-yello}

심지어 위 단계를 다른 객체를 조회하는 경우에도 또 다시 반복해야하니 복잡해질 수밖에 없고, 이는 SQL을 작성하는 개발자에게 힘이 드는 일이 아닐 수 없다.

그렇기 때문에, **DB에 저장할 객체에는 상속관계를 사용하지 않는다.**

### 자바 컬렉션

자바 컬렉션을 사용하면 슈퍼타입 서브타입을 사용하는 것보다 훨씬 간단한 방법으로 객체의 저장과 조회가 가능하다.

- 저장: `list.add(Album);`
- 조회: `Album album = list.get(albumId);` 또는 `Item item = list.get(albumId);`

이렇게 자바 컬렉션을 사용한 객체 관리가 간단한 이유는, 자바 컬렉션 또한 **객체 차원**에서 사용되는 도구이기 때문이다. 즉, 자바 컬렉션은 객체를 데이터베이스에 저장하는 것이 아닌, 메모리 내에서 객체를 저장하고 관리하는 자료 구조이다.


## 연관 관계

![연관관계](https://github.com/lcqff/lcqff.github.io/assets/71930280/a9e1dae8-0359-4535-9634-704f7f7d61a7)


연관 관계의 데이터를 가져오기 위해 객체는 `member.getTeam()`과 같은 **참조**를 사용한다. 그러나 데이터베이스에는 객체와 같은 참조가 없다. 그렇기에 데이터베이스에선 연관관계를 만들기 위해 **외래 키를 사용해 Join**하는 방식을 사용한다.(`JOIN ON M.TEAM_ID = T.TEAM_ID`)

이렇게 객체는 참조를 사용하고, 데이터베이스는 외래키를 사용하는 차이에서 문제가 발생한다.

```jsx
class Member {
	String id; //MEMBER_ID 컬럼 사용
	Long teamId; //TEAM_ID FK 컬럼 사용
	String username;//USERNAME 컬럼 사용
}

class Team {
	Long id; //TEAM_ID PK 사용
	String name; //NAME 컬럼 사용
}
```



객체 저장을 쉽게 하기 위한 객체 구조를 생각해보자. 

**기존의** `Team team;`과 같은 참조 대신 `Long teamId;`라는 외래키를 사용한 것을 확인할 수 있다. 이렇게 참조 대신 외래키를 사용하므로써 **Insert문** 작성이 간단해진다.

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/467e9fea-c827-4f62-a4d0-140354d7b9a5)


그러나 이를 객체다운 모델링이라 할 순 없다: **`Member`** 클래스와 **`Team`** 클래스 사이의 관계가 명시적으로 표현되지 않았다. 즉, **참조**가 빠져있다.

그렇다면 참조를 사용하는 것이 바람직하다. 참조를 사용해 INSERT문을 만들기 위하여 아래와 같은 객체 구조를 생각해볼 수 있다.

```jsx
class Member {
	String id; //MEMBER_ID 컬럼 사용
	Team team; //참조로 연관관계를 맺는다. //**
	String username;//USERNAME 컬럼 사용

	Team getTeam() {
		return team;
	}
}

class Team {
	Long id; //TEAM_ID PK 사용
	String name; //NAME 컬럼 사용
}
```

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/7b16d143-0c52-41cd-a498-c4f2731a18b7)


참조를 사용하니 데이터베이스에 Insert하기가 이전에 비해 까다로워졌다. 그러나 더 문제가 되는 것은 조회이다.

```java
SELECT M.*, T.* FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

public Member find(String memberId) {
	//SQL 실행 ...
	Member member = new Member();
	//데이터베이스에서 조회한 회원 관련 정보를 모두 입력
	Team team = new Team();
	//데이터베이스에서 조회한 팀 관련 정보를 모두 입력

	//회원과 팀 관계 설정
	member.setTeam(team);
	return member;
}
```

생략된 부분이 많으나, 충분히 번거로워 보인다. 

그렇다면 번거로운 부분을 어떻게 줄일 수 있을까? 이 모든 번거로움은 **객체와 데이터베이스간의 불일치**에서 발생하는 일이다.  
그럼 `Member`와 `Team`을 DB가 아닌 자바 컬렉션에 관리한다면? 

### 자바 컬렉션

위에서 언급했듯이, 자바 컬렉션은 **객체 차원**에서 사용되는 도구이기 때문에 DB에서 객체를 관리할 때에 비해 훨신 간결하게 코드를 작성이 가능하다.

- 저장: `list.add(member);`
- 조회: `Member member = list.get(memberId); Team team = member.getTeam();`

### SQL 매핑에서의 해방

객체와 DB간 패러다임 불일치에서 번거로움이 있다면 객체 안에서 객체를 관리하면 되는 일이다. 이는 복잡한 변환 과정을 제거해주고, 자유로운 **객체 그래프 탐색**을 가능하게 해준다. 


<div class="callout">
  <div>💡</div>
  <div>
  	<strong>객체 그래프 탐색</strong>
	<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/529fcad2-5289-49bc-82b3-42b926abb49c">
	자유로운 객체 그래프 탐색을 충족시키기 위해선, 참조를 통해 한 객체에서 다른 객체로 이동하며 전체 객체를 자유롭게 탐색할 수 있어야 한다. <br/> <br/>
	그러나 데이터베이스에선 처음 실행한 SQL문에 따라 탐색 범위에 제한이 걸리게 된다.<br/>
	예를 들어 <code><strong>SELECT M.*, T.* FROM MEMBER M JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID</strong></code> 와 같은 SQ이 있을 때, 탐색 가능한 객체는 Member와 Team뿐이다.<br/>
	이러한 탐색의 제한은 <strong>엔티티 신뢰 문제</strong>를 발생시킨다.
  </div>
</div>

객체 지향 설계가 중요하다고 말들은 하나, 객체답게 모델링 할 수록 SQL로 매핑하는 비용이 늘어나게 된다.  
이런 문제를 해결하기 위해, **객체를 자바 컬렉션에 저장하듯이 DB에 저장**하는 방식이 바로 **JPA(Java Persistence API)**이다.