---
layout: post
title: 디자인패턴(2)- 다형성과 추상 타입
author: pam
categories: Backend-Study
description: "[Study] 구현 변경의 유연함을 위한 추상화"
tags: Java 디자인패턴
toc: true
---

---

이전 포스팅
- [객체지향](https://lcqff.github.io/java/2024/08/19/design-pattern-1.html)  

---

## 개요

객체지향이 주는 가장 핵심적인 장점은 **구현 변경의 유연함**이다.
이러한 유연성을 위한 방법이 두가지 있는데, 하나는 캡슐화이고 다른 하나는 추상화이다.

캡슐화에 대한 내용은 이전 포스팅에서 다뤘으니, 이번엔 추상화에 대해 알아보자. 


## 다형성과 상속

<div class="callout">
  <div>💡</div>
  <div>
    <strong>다형성(Polymorphism)</strong><br/>
    한 객체가 여러가지(poly) 모습(morph) 즉 <strong>타입</strong>을 갖는 것
  </div>
</div>

**자바에서는 타입 상속을 통하여 다형성을 구현**한다. 


```java
public class Plane() {
	public void fly() { }
}
	
public interface Turbo { 
	public boid boost();
}
	
public class TurboPlane extends Plane implements Turbo { 
	public void boost() { }
}
```

위 코드에서는 인터페이스 `Turbo`와 클래스 `Plane`을 상속받고 있는 `TurboPlane`을 확인할 수 있다.

`TurboPlane` 은 Turbo와 Plane에서 **정의된 모든 기능(`fly()`, `boost()`)을 제공**한다.

```java
TurboPlane tp = new TurboPlane();

Plane p = tp; //업캐스팅
Turbo t = tp; //업캐스팅

t.boost();
```

또한 TurboPlane은 Plane 타입과 Turbo 타입이 될 수 있다. 

위 코드는 `TurboPlane`을 업캐스팅을 통해 Plane과 Turbo로 업캐스팅한 예시 코드이다. 이때 변수 p와 t는 모두 동일한 객체인 tp를 가리키고있다. 

`TurboPlane` 객체가 **두 가지 다른 형태(Plane와 Turbo)**로 동작할 수 있다는 점에서 다형성이 발생함을 알 수 있다.

## 인터페이스 상속과 구현 상속

Java에는 세가지 상속이 있다. 이들의 특징에 대해 알아보자. 

### 클래스 상속

```java
public class Plain() {
	public void fly() { 
	// ... 실제 구현
	}
}
	
public class Jet extends Plane { 
}

...

Jet j = new Jet();
j.fly();
```

- **구현 상속**: 상위 클래스에 정의된 기능을 재사용할 수 있다. (상위 클래스에 구현된 메서드의 구현을 함께 상속)
- **단일 상속**: 한 클래스의 하나의 부모 클래스만 상속받을 수 있다. (다중 상속 지원x)
- **재정의**: 하위 클래스는 상위 클래스에 정의된 기능을 자신에 맞춰 수정할 수 있다.

### 인터페이스 상속

```java
public interface Turbo { 
	public void boost();
}
	
public class TurboPlane implements Turbo { 
	public void boost() {
	// ... 실제 구현
	}
}
```

- **시그니처 상속**: 인터페이스는 메서드 시그니처만 제공할 뿐 실제 구현은 제공하지 않는다.
- **다중 상속**: 하나의 클래스가 여러 개의 인터페이스를 상속받을 수 있다.

### 추상 클래스 상속

```java
public abstract class Plain() {
	public abstract void boost();
	
	public void fly() { 
		// ... 실제 구현
	}
	
}
	
public class Jet extends Plane { 
	public void boost() {
		// ... 실제 구현
	}
}

...

Jet j = new Jet();
j.fly();
```

- **단일 상속**: 한 클래스의 하나의 부모 클래스만 상속받을 수 있다. (다중 상속 지원x)
- **추상 메서드**와 **구현된 메서드**를 모두 지님
    - **구현 상속**: 상위 클래스에 구현된 메서드를 재사용할 수 있다.
    - **시그니처 상속**: 추상 메서드는 메서드 시그니처만 제공할 뿐 실제 구현은 제공하지 않는다.

### 메서드 오버라이딩

구현 상속에서는 재정의를 통해 하위 타입이 상위 타입의 메서드 구현을 수정할 수 있다.

이를 <blue>메서드 오버라이딩</blue>이라고 한다. 

```java
public class Plane() {
	public void fly() { 
		// ... 실제 구현
	}
}
	
public class Jet extends Plane { 
	public void (fly) {
		//메서드 오버라이딩을 통한 재정의
	}
}

Plane p = new Jet();
p.fly(); // Jet에서 재정의된 메서드 실행
```

위의 예시를 보자. 하위 타입인 Jet은 상위 타입인 Plane의 메서드를 오버라이딩을 통해 재정의했다.

이때 `p.fly();`을 통해 실행되는 메서드는 Plane의 메서드가 아닌 **Jet에서 재정의된 메서드**이다.

객체 p는 타입캐스팅되었기에 p가 실제로는 타입 Jet을 가리키고 있기 때문이다.

## 추상 타입과 유연함

이하로는 실제 내가 세차새차에서 경험한 추상화의 장점에 대해 설명해보겠다.

<br>

세차새차에서는 현재 예약의 종류를 두 종류로 나누고있다. 고객이 세차새차 플랫폼을 통해 예약을 등록하는 **‘온라인 예약’**과 오프라인으로 예약이 들어올 경우 세차장 사장님이 직접 예약을 등록하는 **‘오프라인 예약’**이다.

현재는 존재하지 않지만 이후 ‘카카오 예약’이나 ‘네이버 예약’과 같은 종류의 예약이 등록될 예정이다.

이때 이 여러 종류의 예약들은 각자 제공하는 기능에 차이점이 있으나 **‘예약 정보를 대시보드에 동일한 형태로 제공함’**이라는 기능만은 모두 동일하게 가지고 있다.

<br>

나는 `OfflineReservation`만을 반영할 수 있는 대시보드가 `OnlineReservation`를 함께 반영시킬수 있도록 기능을 확장하는 이슈를 맡게됐다. 
구현 이전의 코드는 아래와 같다.

```java
public class DashboardManagerService {

	// 생성자 주입

	@Transactional(readOnly = true)
	public DashboardManagerResponseDto getDashboardBySlugWithStatus(String slug) {
		// ... 오늘치 Offline예약을 가져오는 로직
		List<OfflineReservationOrder> todayOrders = offlineReservationOrderService.getAll(todayOfflineReservationList);

		return DashboardManagerResponseDto.builder()
			//오늘치 Offline 예약의 총 개수
			.reservationCount(todayOfflineReservationList.size())
			//오늘 미완수된 Offline 예약의 총 개수
			.remainingReservationCount(dashboardHelper.calculateRemainingReservationCountWithStatus(todayOfflineReservationList))
			//오늘치 Offline 예약 수익
			.todayRevenue(dashboardHelper.calculateTodayRevenueWithStatus(todayOfflineReservationList))
			//오늘 가장 많이 주문된 Offline 예약 메뉴
			.mostOrderedMenuName(dashboardHelper.findMostOrderedMenuNameWithStatus(todayOrders))
			.offlineReservations(...)
			.build();
	}
}
```

```java
public class DashboardHelper {

	//오늘치 Offline 예약 수익
	@Transactional(readOnly = true)
	public Integer calculateTodayRevenueWithStatus(List<OfflineReservation> todayOfflineReservationList) {
		return todayOfflineReservationList.stream()
			.filter(reservation -> OfflineReservationPaymentStatus.PAID.equals(reservation.getPaymentStatus())) // 결제 완료된 예약만 필터링
			.mapToInt(OfflineReservation::getTotalPrice)
			.sum();
	}

 //오늘 가장 많이 주문된 Offline 예약 메뉴
	@Transactional(readOnly = true)
	public String findMostOrderedMenuNameWithStatus(List<OfflineReservationOrder> todayOfflineReservationOrderList) {
		return todayOfflineReservationOrderList.stream()
			.filter(order -> order.getParentOrder() == null && OfflineReservationStatus.CONFIRMED.equals(order.getOfflineReservation().getReservationStatus()))  // 옵션 제외, 확정된 예약만 필터링
			.collect(Collectors.groupingBy(OfflineReservationOrder::getMenuName, Collectors.counting()))  // 메뉴 이름 별 개수
			//...
			.orElse("");
	}

 //오늘 미완수된 Offline 예약의 총 개수
	@Transactional(readOnly = true)
	public Integer calculateRemainingReservationCountWithStatus(List<OfflineReservation> todayOfflineReservationList) {
		return (int)todayOfflineReservationList.stream()
			.filter(reservation -> OfflineReservationStatus.CONFIRMED.equals(reservation.getReservationStatus())  // 확정된 예약만
					&& !OfflineReservationPaymentStatus.PAID.equals(reservation.getPaymentStatus())  // 결제 완료된 예약 제외
					&& !reservation.isNoShow())  // 노쇼 제외
			.count();
	}
}
```

대시보드에 제공할 형태를 가공하는 역할인 DashboardHelper의 모든 메서드가 OfflineReservation 용으로 fit하게 작성돼있다.

OnlineReservation은 DashboardHelper의 모든 메서드 `calculateTodayRevenueWithStatus`,  `findMostOrderedMenuNameWithStatus`, `calculateRemainingReservationCountWithStatus`을 사용할 수 없으므로  OnlineReservation용의 기능을 개발하기 위해서는 이미 존재하는 메서드들과 동일한 구현을 가진 메서드를 새로 작성해야하는 상황이다.

<br>

```java
public class DashboardHelper {

	//오늘치 Offline 예약 수익
	@Transactional(readOnly = true)
	public Integer calculateTodayRevenueWithStatus(List<OfflineReservation> todayOfflineReservationList) {
	// ...
	}

	//오늘 가장 많이 주문된 Offline 예약 메뉴
	@Transactional(readOnly = true)
	public String findMostOrderedMenuNameWithStatus(List<OfflineReservationOrder> todayOfflineReservationOrderList) {
	// ...
	}

	//오늘 미완수된 Offline 예약의 총 개수
	@Transactional(readOnly = true)
	public Integer calculateRemainingReservationCountWithStatus(List<OfflineReservation> todayOfflineReservationList) {
	// ...
	}

	//오늘치 Online 예약 수익
	@Transactional(readOnly = true)
	public Integer calculateTodayRevenueWithStatus(List<OnlineReservation> todayOnlineReservationList) {
		return todayOnlineReservationList.stream()
			.filter(
					//... OnlineReservation용의 구현
					) 
			.mapToInt(OnlineReservation::getTotalPrice)
			.sum();
	}

	//오늘 가장 많이 주문된 Online 예약 메뉴
	@Transactional(readOnly = true)
	public String findMostOrderedMenuNameWithStatus(List<OnlineReservationOrder> todayOnlineReservationOrderList) {
		return todayOnlineReservationOrderList.stream()
			.filter(
					//... OnlineReservation용의 구현
					) 
			.collect(Collectors.groupingBy(OnlineReservationOrder::getMenuName, Collectors.counting()))  // 메뉴 이름 별 개수
			//...
			.orElse("");
	}

	//오늘 미완수된 Online 예약의 총 개수
	@Transactional(readOnly = true)
	public Integer calculateRemainingReservationCountWithStatus(List<OnlineReservation> todayOnlineReservationList) {
		return (int)todayOnlineReservationList.stream()
			.filter(
					//... OnlineReservation용의 구현
					) 
			.count();
	}
}
```

위와 같은 코드는 그 누구도 바라지 않는다.

### 추상 타입을 이용한 구현 교체의 유연함

나는 위와 같은 코드의 문제를 **다형성**을 이용해 해결했다.

```java
public interface ReservationOrder {
	Integer getTotalPriceOfPaidOrder();

	Boolean isMenuAndStatusConfirmed();

	Boolean isStatusConfirmedAndPaidWithoutNoShow();

	String getMenuName();
}

```

```java
public OnlineReservation implements ReservationOrder {
	// ...
}

public OfflineReservation implements ReservationOrder {
	// ...
}
```

추상 타입 ReservationOrder의 상속에 의해 `OnlineReservation`과 `OfflineReservation`은 다형성에 의해 **자기 자신의 타입뿐만 아니라 `ReservationOrder`의 타입으로도 동작**하게 된다.

이러한 다형성의 특징을 사용하여 위의 코드를 아래와 같이 다시 구현할 수 있다.

<br>

```java
public class DashboardManagerService {

	// 생성자 주입

	@Transactional(readOnly = true)
	public DashboardManagerResponseDto getDashboardBySlugWithStatus(String slug) {
		// ... 오늘치 Offline 예약과 Online 예약 리스트를 가져오는 로직
		List<OfflineReservationOrder> todayOfflineOrders = offlineReservationOrderService.getAll(todayOfflineReservations);
		List<OnlineReservationOrder> todayOnlineOrders = onlineReservationOrderService.getAll(todayOnlineReservations);
		List<ReservationOrder> todayReservationOrders = new ArrayList<>();
		todayReservationOrders.addAll(todayOfflineOrders);
		todayReservationOrders.addAll(todayOnlineOrders);

		return DashboardManagerResponseDto.builder()
			//오늘치 Offline 예약의 총 개수
			.reservationCount(todayReservations.size())
			//오늘 미완수된 예약의 총 개수
			.remainingReservationCount(dashboardHelper.calculateRemainingReservationCountWithStatus(todayReservationOrders))
			//오늘치 예약 수익
			.todayRevenue(dashboardHelper.calculateTodayRevenueWithStatus(todayReservationOrders))
			//오늘 가장 많이 주문된 예약 메뉴
			.mostOrderedMenuName(dashboardHelper.findMostOrderedMenuNameWithStatus(todayReservationOrders))
			.offlineReservations(...)
			.onlineReservations(...)
			.build();
	}
```

```java
public class DashboardHelper {
	
	@Transactional(readOnly = true)
	public Integer calculateTodayRevenueWithStatus(List<ReservationOrder> reservationOrders) {
		return reservationOrders.stream()
			.mapToInt(ReservationOrder::getTotalPriceOfPaidOrder)
			.sum();
	}

	@Transactional(readOnly = true)
	public String findMostOrderedMenuNameWithStatus(List<ReservationOrder> reservationOrders) {
		return reservationOrders.stream()
			.filter(ReservationOrder::isMenuAndStatusConfirmed)
			.collect(Collectors.groupingBy(ReservationOrder::getMenuName, Collectors.counting()))  // 메뉴 이름 별 개수
			. // ...
			.orElse("");
	}

	@Transactional(readOnly = true)
	public Integer calculateRemainingReservationCountWithStatus(List<ReservationOrder> reservationOrders) {
		return (int) reservationOrders.stream()
			.filter(ReservationOrder::isStatusConfirmedAndPaidWithoutNoShow)
			.count();
	}
}
```

다형성을 통해 이제 ReservationOrder 타입을 상속하는 모든 예약은  DashboardHelper의 모든 메서드 `calculateTodayRevenueWithStatus`,  `findMostOrderedMenuNameWithStatus`, `calculateRemainingReservationCountWithStatus` 를 수정 없이 사용할수 있다.

이는 다형성을 통해 얻을 수 있는 유연성의 예시이다.

- ReservationOrder의 종류가 변경되어도 DashboardHelper 클래스는 변경되지 않는다.

<br>

이때 상위 타입 `ReservationOrder`의 기능을 실제로 구현하는 하위 클래스(ex. OnlineReservation, OfflineReservation)를 실제 구현을 제공한다는 의미에서  <red> ‘콘크리트 클래스’</red> 라고 부른다.

### 규칙: 인터페이스에 대고 프로그래밍하기

<div class="callout">
  <div>💡</div>
  <div>
    <strong>인터페이스에 대고 프로그래밍하기 (program to interface)</strong><br/>
    콘크리트 클래스가 아닌, 기능을 정의한 인터페이스를 사용해서 프로그래밍하라는 규칙
  </div>
</div>

위는 객체 지향의 유명한 규칙 중 하나이다. 

여기서 말하는 인터페이스란 Java와 같은 프로그래밍 언어에서 제공하는 인터페이스가 아닌, 오퍼레이션을 정의한 인터페이스를 의미한다.

*program to interface은* 추상화를 통한 유연성을 얻기 위한 규칙이다. 

이 원칙의 핵심은 코드에서 **구체적인 구현 클래스 대신 인터페이스나 추상 클래스를 사용하여 객체와 상호작용하는 것**이다. 

그러나 주의할 점은, 추상타입의 증가는 구조의 불필요한 복잡함을 야기한다는 점이다. 따라서 우리는 <red>변화 가능성이 높은 경우에만 인터페이스를 사용</red> 해야만 한다.