---
layout: post
title: 디자인패턴(1)- 객체 지향
author: 팜
categories: Java
description: \[Study] 객체 지향적 설계를 위해 지켜야하는 규칙 공부 
tags: Java 디자인패턴
sidebar:
---

## 0. 개요
본 포스팅은 '개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴'을 읽고 정리한 포스팅이다. 
디자인 패턴을 공부하게 된 계기는 단순하다. '어떻게 코드를 짜야 좋은 코드가 되는가' 고민하는 것을 넘어 이를 같이 일하는 팀원에게 설득하려면, 단순 나의 뇌피셜이 아닌 신뢰할 수 있는 자료나 기술이 뒷받침되야 할거 같다 느꼈기 때문이다.😅

## 1. 절차지향과 객체지향

<div class="callout">
  <div>💡</div>
  <div>
    <strong>절차지향</strong><br/>
    프로시저(procedure)을 통해 프로그램을 구성하는 기법
  </div>
</div>

절차지향은 **Procedure Oriented**의 번역이다.

여기서 Procedure가 의미하는 바를 살펴보면 아래와 같다.

#### 사전적 Procedure

![image](https://github.com/user-attachments/assets/4f1527e9-5872-4294-9d0c-9d4367f82e79)

사전상 Procedure의 의미는 위와 같다.

#### IT의 프로시저

```sql
DELIMITER $$

CREATE PROCEDURE GetEmployeesByDepartment(IN dept_name VARCHAR(50))
BEGIN
    SELECT employee_id, first_name, last_name
    FROM employees
    WHERE department = dept_name;
END $$

DELIMITER ;

```

동시에 개발 용어로서의 프로시저는 **데이터베이스에 대한 일련의 작업을 정리한 절차를 관계형 데이터베이스 관리 시스템이 저장한 것**을 의미한다. 
위와 같은 식은 컴퓨터 공학을 전공했거나 백엔드 공부를 해본 사람이라면 익숙할지도 모른다. 위와 같은 식이 바로 <blue>프로시저</blue>이다.


![image](https://github.com/user-attachments/assets/12418ee1-1925-47d7-a950-5f5f1a1b8195)

절차지향 방식은 이런 **프로시저를 사용하여 데이터를 조작하는 코드**를 작성하는 것이다. 그래서 개인적으로 ‘절차’보다는 ‘데이터 중심적인 코드 작성 방법’이라 생각하는 편이 더 이해가 빨랐다.


![image](https://github.com/user-attachments/assets/2efd8cc0-fcd1-47c9-be1e-03e5f420a559)
<cap>평균값 출력 프로그램</cap><br>

위는 수학,영어,국어 점수의 평균값을 계산한 후 평균값을 화면에 출력하는 프로그램의 구조이다. 이렇게 절차 지향 프로그램은 데이터 중심으로 짜여지고, 자연스럽게 프로시저끼리 공유하는 데이터가 생긴다. 

### 절차지향 방식의 문제

그러나 이런 ‘데이터 중심적인’ 방식은 데이터 의존도에 의해 문제가 발생한다.

- 데이터 타입이나 의미가 변경되면 함께 수정해야하는 프로시저가 증가한다.
- 같은 데이터를 프로시저들이 다른 의미로 사용하는 경우가 발생한다.


![image](https://github.com/user-attachments/assets/1d99025c-7add-4a7e-a318-9dae315bae76)


예를 들어 위의 평균값 출력 프로그램에 ‘시험종료’ 데이터를 추가해보자. 

시험 종료 데이터는 Boolean 값으로, true일때 평균 계산 프로시저와 화면 출력 프로시저가 작동한다. 

그러나 ‘시험 종료’ 데이터를 ‘시험 시작 전’, ‘수학 시험 종료’, ‘영어 시험 종료’, ‘국어 시험 종료’, ‘시험 종료’ 타입을 지닌 **Enum**값으로 변경한다고 하자. 

‘시험 종료’ 데이터의 수정은 ‘시험 종료’ 데이터를 공유하는 ‘평균 계산 프로시저’와 ‘화면 출력 프로시저’의 수정을 야기한다. 

## 2. 객체 지향

<div class="callout">
  <div>💡</div>
  <div>
    <strong>객체 지향</strong><br/>
    객체를 통해 프로그램을 구성하는 기법
  </div>
</div>

객체 지향은 데이터와 데이터와 관련된 프로시저를 객체(Object)라는 단위로 묶는 것으로 시작한다. 

![image](https://github.com/user-attachments/assets/c5828a16-4968-4ed1-951d-607576c4fa74)

객체는 자신만의 기능을 제공하며, 각 객체들은 서로 연결되어 다른 객체가 제공하는 ‘프로시저’를 사용할 수 있다.

이때 한 객체의 프로시저는 자신이 속한 객체의 데이터만 접근 가능하다.

{: .box-yello}
- 초기 설계가 까다롭다
- 데이터가 변경되더라도 해당 객체 내부에서만 변화가 집중되고 다른 객체에는 영향을 주지 않는다. (캡슐화)

## 3. 객체

‘객체’를 정의할때 사용하는 용어에 대해 알아보자.

객체의 사용은 데이터가 아닌 ‘기능’에 집중된 구현을 가능하게 해준다. 따라서 객체는 객체가 제공하는 기능(책임)으로 정의된다. 이때 객체가 제공하는 기능을 <blue>오퍼레이션(Operation)</blue>이라 한다. 


또한 이런 오퍼레이션을 사용하기 위해서는 오퍼레이션의 사용 방법을 알아야만 한다. 이를 <blue>‘시그니처(Signature)’</blue>라 하며, 시그니처는 ‘**식별 이름, 파라미터 및 파라미터 타입, 리턴 값**’의 3가지로 구성된다.


객체가 제공하는 오퍼레이션의 집합을 <blue>‘인터페이스(Interface)’</blue>라 한다. (이때 인터페이스는 Java에서 제공하는 Interface와는 다른 개념이다.) 인터페이스를 구분할 때는 <blue>타입(type)</blue>이라는 명칭으로 구분한다. 


인터페이스는 기능에 대한 명세서 역할을 하며 한 객체가 지닌 책임을 정의한다. 


마지막으로 <blue>클래스(class)</blue>는 실제 객체가 기능을 어떻게 구현하는지에 대한 정보이다. 이때 class는 Java에서 사용하는 Class와 유사하다.


‘소리 크기 제어 객체’ 예시를 통해 위 용어를 알아보자. 이편이 이해가 빠를 것이다.

‘소리 크기 제어 객체’는 ‘소리 증가’, ‘소리 감소’, ‘음소거’의 3가지 기능을 지니고 있다.



![image](https://github.com/user-attachments/assets/5086238e-4809-4af3-af11-3380e2c83583)

### 객체의 책임

상황에 따라 객체의 책임 구성은 달라질 수 있다. 그러나 확실한 규칙이 있다면 그것은 ‘객체의 책임, 즉 객체가 제공하는 기능은 작을수록 좋다’이다.


![image](https://github.com/user-attachments/assets/3270d855-01bc-45a2-921d-12a4a375537a)

위 이미지는 동일한 기능을 절차지향적으로 구현한 것과(좌), 모든 기능을 한 객체에 모두 밀어넣은 것이다(우).

사실상 두 구현에는 큰 차이가 없다. 객체지향적인 구현을 위해 **객체를 사용했음에도 절차지향적인 구조를 지니게 되는 것**이다. 이는 절차지향의 가장 큰 문제인 기능 변경의 어려움이 나타날 수 있음을 의미한다.

<br>

자, 여기서 옛날 이야기를 하나 해보겠다. 내가 ‘두레’ 프로젝트를 진행하며 생긴 일이다…


![image](https://github.com/user-attachments/assets/5143de4c-d8f6-49e8-984f-3766fc387a8d)

 위 이미지는 내가 진행하고 있는 두레의 코드 일부이다.

위 이미지를 잘 보면 `validateExistMember` 코드가 회원 검증이 필요한 모든 도메인의 Service코드에 private으로 작성되어있음을 확인 할 수 있다. 

당연히 위와 같이 작성해도 코드는 정상적으로 돌아간다. 그러나 문제는 다음부터 발생한다.

두레에는 `validateExistMember`을 통한 회원 검증뿐만이 아니라, 기타 다른 검증요소 또한 private으로 작성되어 있었다. 예를 들어 팀에서 팀원의 Role을 가져오고, 만일 Role이 없다면 팀원이 아닌 것으로 판단하는 팀원 검증 코드와 같은…

```java
private Map<Member, TeamRoleType> getRoleOfMember(final Long teamId, final List<Member> members) {
	return members.stream()
					.collect(Collectors.toMap(
		          member -> member,
	            member -> teamRoleRepository.findTeamRoleByTeamIdAndMemberId(teamId, member.getId())
	              .orElseThrow(() -> new MemberException(NOT_FOUND_MEMBER))
                .getTeamRoleType()
           ));
}
```

그러나 여기서 문제가 발생한다. 검증코드를 작성하던 개발자가 실수로 

`teamRoleRepository.findTeamRoleByTeamIdAndMemberId(teamId, member.getId())`로 작성해야 할 코드를 `teamRoleRepository.findTeamRoleByTeamId(teamId);`로 작성해버렸다. 

거기에 다른 팀원들 또한 이를 확인하지 못하고 approve하여 코드가 그대로 메인 브랜치에 머지되어 버렸다!

실수를 알아차린 것은 그로부터 한참 뒤였다.


그러면 이제 무엇을 해야하는가? 사실 위 문제를 해결하는 건 어렵지 않다. 팀원 검증 로직의`findTeamRoleByTeamId`을 `findTeamRoleByTeamIdAndMemberId`로 수정해주면 되는 일이기 때문이다.

그러나 private으로 선언된 팀원 검증 코드는 두레의 모든 도메인에 뿌려져있었고…. 이를 모두 찾아 하나 하나 수정하고 문제 없이 수정되었는지 확인 하기 위해서는 꽤 많은 시간이 필요했다.

<br>

위 문제의 원인은 **책임 분리가** 적절하게 이루어지지 않았기 때문이다. 두레에서는 ‘팀 관리’, ‘스터디 관리’, ‘회원 관리’와 같은 기능들에 ‘검증’의 책임이 적절하게 분리되지 않았다. 
만일 개발자들이 ‘검증’의 책임을 검증 Service로 분리하고, ‘팀 관리’, ‘스터디 관리’, ‘회원 관리’와 같은 기능은 검증 Service에서 해당 기능을 가져와 사용하는 식으로 개발하였으면  검증 Service의 코드 단 한줄만을 수정하는 것으로 모든 문제가 해결되었을 것이다.

 이렇게 객체의 책임과 관련된 원칙으로 <red>단일 책임 원칙 (Single Responsibility Principle: SRP)</red>이 존재한다. 

이는 한 객체는 단 한개의 책임만을 지녀야 한다는 원칙이다. 

### 객체의 의존

객체의 의존은 필연적으로 발생한다. (위 이미지를 예시로 들면, 한곳에 몰아넣어져 서로 교류하던 기능들을 서로 다른 객체로 분리한 것이니 의존이 존재하지 않을 수가 없다.)


![image](https://github.com/user-attachments/assets/a07a57b7-6919-44cc-9fae-e9dad45d36a9)

위의 예시를 그대로 가져오자면, 흐름 제어 객체는 파일읽기 객체, 파일 쓰기 객체, 암호화 객체에 대한 의존성을 지닌다.


<div class="callout">
  <div>💡</div>
  <div>
    <strong>의존(Dependency)</strong><br/>
    한 객체가 다른 객체를 생성하거나, 파라미터로 전달받거나, 다른 객체의 메서드를 호출하는 것.<br>
    한 객체의 변경사항이 다른 객체의 변경을 줄 가능성이 높아지는 것.
  </div>
</div>

![image](https://github.com/user-attachments/assets/5f8b273a-3739-47e2-87ba-5f1166a881f3)

의존은 전이된다. 위 이미지는 Controller가 ServiceA에 의존하고, ServiceA는 ServiceB에 의존하고 있는 상황을 나타낸다. 

이때 ServiceB의 변경은 ServiceA의 변경을 야기시킬 가능성이 크다. 또한 ServiceA의 변경은 Controller의 변경을 야기시킬수 있다. 이를 **의존의 전이**라고 한다.

이런 의존의 특징에 의해 나타날수 있는 문제가 바로 **순환 의존**이다.

![image](https://github.com/user-attachments/assets/107d5e34-d878-4ac5-bcc5-6c527f4eba53)

자기 자신의 변경이 다시 자시 자신의 변경을 유발할 수 있다는 것이다.

### Springboot에서의 순환 참조 문제

Spring을 사용하여 개발을 하다보면 Service에서 다른 Service의 메서드를 필요로 하여, Service간 의존을 하게되는 상황을 자주 마주하게 될 것이다. 이때도 순환참조의 문제가 발생할 수 있다. 

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final OrderService orderService;    
    ...
}
```

```java
@Service
@RequiredArgsConstructor
public class OrderService {
	private final UserService userService;

	   public List<Order> findAllByUserServiceId(Long id) {
        User user = userService.findById(id);
        return orderRepository.findAllByUser(user);
    }
    ...
}
```

예시로 위와 같은 코드가 있다.

위와 같은 구조는 `BeanCurrentlyInCreationException`에러를 발생시킨다. 빈을 초기화 시킬 때 `UserService`의 빈을 생성하기 위해서는 `OrderService`의 빈이, `OrderService`의 빈을 생성하기 위해서는 `UserService`의 빈이 필요한데 **두 빈 중 그 어느 것도 생성될 수 먼저 생성될 수 없기에 발생하는 문제**이다. 

<gray>그러나 경험상 순환참조를 의식하지 않고 설계를 해도 순환참조가 발생하는 일은 없었다. 설계만 잘 돼있다면 순환참조에 대해 걱정할 필요는 없을 듯…</gray>

이런 순환 의존이 발생하지 않도로 하는 원칙 중 하나가 <red>의존 역전 원칙(DIP: Dependency inversion principle)</red>이다. 

## 4. 캡슐화


<div class="callout">
  <div>💡</div>
  <div>
    <strong>캡슐화</strong><br/>
    객체의 기능 구현을 감춰 내부 기능 구현이 변경되더라도 해당 기능을 사용하는 코드에 대한 영향을 줄이는 것.
  </div>
</div>

```java
if (member.isMale() && member.getExpriyDate().getDate() < currentDate) {
	//만료 되었을 때의 처리
}
```

```java
if (member.isExpried()) {
	//만료 되었을 때의 처리
}
```

캡슐화의 가장 쉬운 예시로는 위와 같은 예시가 있다. 

구체적인 로직을 `isExpried()`로 캡슐화 시킴으로써 우리는 그 내부 코드를 알 필요가 없게되었다. 

객체의 책임에 대해 설명했을 때 예시로 든 검증 코드또한 캡슐화의 예시가 될 수 있겠다. 


캡슐화 되지 않은 코드는 변경이 필요해질 때, 해당 코드가 사용되는 곳을 모두 찾아 수정해주어야 하며, 해당 코드를 사용하는 곳이 많을수록 수정에서 실수가 발생할 가능성이 높아진다. 아니, 해당 코드들이 사용되는 곳을 찾는 것부터가 문제이다. 

이는 코드가 데이터 중심으로 짜여졌기 때문이다. 데이터를 직접적으로 사용하기에 데이터의 변화에 영향을 받는 코드들이 연쇄적으로 발생하는 것이다.

그러나 캡슐화를 사용하면 캡슐화된 코드를 수정하는 것만으로 기능 수정이 가능하며, 반환값이 수정되지 않는 이상 연쇄적으로 영향을 받는 코드 또한 발생하지 않는다.


아래는 캡슐화를 돕기위한 두가지 법칙이다.

### Tell, Don’t Ask

Tell, Don’t Ask 규칙이란 데이터를 물어보지 않고 **기능의 실행만을 요청하는 것**이다.

```java
if(member.getExpiryDate().getDate() < currentDate {
	//만료 되었을 때의 처리
}
```

```java
if(member.isExpired()) {
	//만료 되었을 때의 처리
}
```

### 데미테르의 법칙

데미테르의 법칙은 Tell, Don’t Ask규칙을 지킬수 있도록 도와주는 규칙이다. 데미테르의 법칙은 아래 세가지 규칙으로 이루어진다.

- 메서드에서 생성한 객체의 메서드만 호출한다.
- 파라미터로 받은 객체의 메서드만 호출한다.
- 필드로 참조하는 객체의 메서드만 호출한다.

이를테면 아래와 같은 예시가 있다.

```java
Wallet wallet = custuomer.getWallet();
if (wallet.getTotalMoney() >= payment) {
	wallet.substractMoney(payment);
}
```

위 코드는 Custuomer 객체의 getWallet() 메서드를 통해 생성한 Wallet 객체에서 또다시 getTotalMoney() 메서드를 호출한다.

데미테르의 법칙을 따르기 위해선 Custuomer 객체에 대한 단 한번의 호출만으로 TotalMoney을 받아올 수 있어야 한다.

```java
int paidAmount = customer.getPayment(payment);
```

 

## 5. 객체지향 설계과정

위에서 살펴본 내용을 종합적으로 정리하자면 객체지향적인 설계를 위해선 다음과 같은 과정을 필요로 한다.

{: .box-note}
1. 필요한 기능을 찾고 세분화한다. 
2. 세분화 한 기능을 알맞은 객체에 할당한다.
    1. 기능구현에 필요한 데이터를 객체에 할당한다. 
    2. 기능은 최대한 캡슐화해서 구현한다.
3. 객체간 메세지를 주고받는 방법을 결정한다. 
4. 위를 계속 반복한다.