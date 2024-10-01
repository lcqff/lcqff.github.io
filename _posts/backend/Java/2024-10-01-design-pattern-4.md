---
layout: post
title: 디자인패턴(4)- 설계 원칙 SOLID
author: 팜
categories: Java
description: \[Study] 객체 지향 설계를 위한 SOLID 원칙에 대해 학습
tags: Java 디자인패턴
sidebar:
---

이전 포스팅
- [객체지향](https://lcqff.github.io/java/2024/08/19/design-pattern-1.html)  
- [다형성과 추상 타입](https://lcqff.github.io/java/2024/09/24/design-pattern-2.html)  
- 

---
## 0. 개요

앞서 객체지향의 기본이 되는 책임 할당, 캡슐화, 다형성과 추상화, 조립을 통한 재사용에 대해 알아봤듯이 객체지향 설계는 소프트웨어의 변경을 유연하도록 돕는다.

**SOLID**는 이러한 객체 지향적인 설계를 돕는 설계 원칙이다. 

SOLID는 앞서 살펴본 책임 할당, 캡슐화, 다형성과 추상화, 조립을 통한 재사용과 같은 내용들을 체계적으로 정리한 것이다.


{: .box-yello}
- 단일 책임 원칙 (Single responseibility principle: SRP)
- 개방-폐쇄 원칙 (Open-closed principle: OCP)
- 리스코프 치환 원칙 (Liskov subsitution principle: LSP)
- 인터페이스 분리 원칙 (Interface segregation principle: ISP)
- 의존 역전 원칙 (Depencency inversion principle: DIP)

## 1. 단일 책임 원칙

> 클래스는 단 한 개의 책임을 가져야 한다.
> 
> 즉, 클래스를 변경하는 이유는 단 한개여야 한다.

<br> 

```java
public class DataViewer {
	
	public void display() {
		String data = loadHtml();
		updateGui(data);
	}
	
	public String loadHtml() { //HTTP 응답을 반환한다.
		HttpClient client = new HttpClient();
		client.connect(url);
		return client.getResponse();
	}
	
	private void updateGui(String data) { // HTML 문자열을 GUI형태로 보여준다. 
		GuiData guiModel = parseDataToGuiData(data);
		tableUI.changeData(guiModel);
	}
		
	private GuiData parseDataToGuiData(String data) {
		// ...
	}
	
	// ...
}
```
<cap>HTTP 프로토콜을 사용하여 데이터를 읽어와 화면에 보여주는 클래스</cap><br>

DataViewer클래스는 HTTP 응답을 받을 뿐만 아니라, 이를 화면에 보여주는 두 가지 책임을 지니고 있다.

그렇다면 만일 서버에서 제공하는 데이터가 HTTP가 아닌 다른 형태로 바뀐다면 어떻게 될까?

### 코드 수정의 어려움

```java
public class DataViewer {
	
	public void display() {
		byte[] data = load();
		updateGui(data);
	}
	
	public byte[] loadHtml() { // 로직 변경
		SocketClient client = new SocketClient();
		client.connect(server, port);
		return client.read();
	}
	
	private void updateGui(byte[] data) { // 변경 발생
		GuiData guiModel = parseDataToGuiData(data);
		tableUI.changeData(guiModel);
	}
	
	private GuiData parseDataToGuiData(byte[] data) { //변경 발생
		// ...
	}
	
	// ...
}
```
<cap>HTTP → Socket으로 응답 데이터가 수정되었다.</cap><br>

HttpClient를 SocketClient로 교체한 것만으로도 해당 데이터를 사용하는 모든 메서드에 변경이 발생되었다. 

이러한 문제는 하나의 클래스가 데이터를 읽는 책임과 화면에 보여주는 책임, **두가지의 책임을 지니고 있기 때문**이다. 

한 클래스가 지니는 책임의 개수가 많아질수록 한 책임의 기능 변화가 다른 책임에 주는 영향은 비례해서 증가한다. 즉, 코드를 절차지향적으로 만든다. 

### 재사용의 어려움

![image](https://github.com/user-attachments/assets/4a472055-186b-41d1-932b-6390ddcc76e1)


`DataViewer`가 `HttpClient`와 `GuiComp`라는 두 패키지를 사용한다고 하자. 또한 데이터를 읽어오는 기능이 필요한 `DataRequireClient`가 `DataViewer`을 사용하고 있다. 

이때 DataRequireClient가 필요한 것은 `DataViewer`와 `HttpClient Jar 파일`이나, DataViewer는 GuiComp를 사용하고 있으므로 실제로는 **DataViewer와 HttpClient Jar 파일 뿐만 아니라 GuiComp의 Jar 파일까지 필요하게 된다.**

<br>


![image](https://github.com/user-attachments/assets/de076c98-e8c7-42d2-872f-e541b234dd1b)

만일 단일 책임 원칙을 적용시켜 책임을 분리한다면 `DataRequireClient`는 `DataLoader`와 `HttpClient 패키지`만을 사용할 수 있게 된다. 

### 책임이란

객체의 존재 이유란, 책임을 할당하기 위함이다. ‘단일 책임 원칙'에서 우리는 한 객체는 하나의 책임만을 지녀야 한다는 개념을 배웠다.

그러나 책임이란 것은 어떻게 구분하면 좋단 말인가?

<br>

![image](https://github.com/user-attachments/assets/2d4987a9-469a-49a4-9bc0-0ccb6cad6e14)

위 이미지에서 GUIApplication은 DataViewer의 `display()` 메서드를 사용하고 DataProcessor는 `loadData()`를 사용한다. 

만일 <blue>GUIApplication에서 요구되는 변경사항이 발생할 경우 DataViewer의 display()가 변경될 확률이 높다.</blue>

또한 <blue>DataProcess에서 요구되는 변경사항이 발생할 경우 DataViewer의 loadData()가 변경될 확률이 높아진다.</blue>

이렇게 **서로 다른 클래스가 서로 다른 메서드를 사용하는 경우**, 이 메서드들은 각기 다른 책임을 지니고 있을 확률이 높다.

## 2. 개방 폐쇄 원칙

> 확장에는 열려 있어야하고, 변경에는 닫혀 있어야 한다.
>  
> 즉, 기능을 변경하거나 확장할 수 있으면서 그 기능을 사용하는 코드는 수정하지 않는다.

<br>

기능을 변경하면서도 코드를 수정하지 않는다는 말이 생소하게 들릴 수 있으나, 이전에 한번 알아보았던 내용이다.

[다형성과 추상 타입](https://lcqff.github.io/java/2024/09/24/design-pattern-2.html)에서 우리는 여러 종류의 Reservation을 DashboardHelper을 사용하여 출력하는 법에 대해 알아보았다.

### 인터페이스를 통한 개방 폐쇄 원칙

![image](https://github.com/user-attachments/assets/558e21f1-581e-4c52-a65e-24f25d699e89)


위와 같은 구조의 코드에서 `DashboardHelper`에 다른 Reservation을 추가해야하는 경우가 생기더라도, 우리는 코드의 수정 없이 <red>ReservationOrder를 상속받는 Reservation 클래스를 새로 만드는 것으로 기능 추가</red> 를 할 수 있었다.

즉, **기능을 확장하면서도 기존의 코드에 대한 변경은 발생하지 않는다**.

이러한 개방 폐쇄 원칙을 구현할 수 있는 이유는 **변화되는 부분(위 그림에서는 Reservation을 읽어오는 부분)을 추상화** 했기 때문이다.

### 상속을 통한 개방 폐쇄 원칙

```java
public class ResponseSender {
	private Data data;
	public ResponseSender (Data data) {
		this.data = data
	}
	
	public void send() {
		sendHeader();
		sendBody();
	}	
	
	protected void sendHeader() {
		// 헤더 데이터 전송
	}	
	
	protected void sendBody() {
		// 바디 데이터 전송
	}
}
```

위 예시는 HTTP 응답 데이터를 전송하는 클래스인 ResponseSender이다.

눈여겨봐야 할 점은  `sendHeader()`와 `sendBody()`가 **protected로 선언**되어있다는 점이다.

```java
public class ZippedResponseSender extends ResponseSender {
	public ZippedResponseSender(Data data) {
		super(data);
	}
	
	@Overried
	protected void sendBody() {
		//데이터 압축 처리
	}
}
```

HTTP 응답 데이터를 전송할 뿐만 아니라, **응답 데이터를 압축해서 전송할 수 있도록 기능을 확장**하려 한다.

이를 위해서 ResponseSender의 `sendBody()`를 오버라이드하여 압축 로직을 추가한 ZippedResponseSender클래스를 생성했다. 

이를 통해 기존 코드에 변경을 일으키지 않은 채, 압축 기능을 추가할 수 있다.

### 개방 폐쇄 원칙의 위반

개방 폐쇄 원칙을 지키지 못한 코드에서는 여러가지 특징을 확인할 수 있다. 예시를 통해 이러한 특징을을 알아가보자.

#### 다운 캐스팅을 한다, instanceof을 사용한다.

![image](https://github.com/user-attachments/assets/29ae8803-4ce9-4214-a178-da9dbe959448)

위와 같은 상속관계를 지니는 슈팅게임을 개발하려 한다. 이때, 화면에 캐릭터를 표시하는 코드가 아래와 같이 구현되었다.

```java
public void drawCharacter(Character character) {
	if (character instanceof Missile) {
		Missile missile = (Missile) character; //다운캐스팅
		missile.drawSpecific();
	} else {
		character.draw();
	}
}
```
<cap>화면에 캐릭터를 표시하는 코드</cap><br>

만일 Character 타입이 Missile인 경우, 별도로 `drawSpecific()` 메서드를 호출하도록 처리하고 있다. 

이러한 구현은 Character 클래스가 확장될 때 `drawCharacter()` 메서드가 함께 수정될 가능성을 증가시킨다.

즉 변경에 닫혀있지 않다.

#### 비슷한 if-else 블록이 존재한다.

위의 예시에서, Enemy 캐릭터의 움직이는 경로를 패턴화하는 코드를 작성하려 한다.

```java
public class Enemy extends Character {
	private int pathPattern;
	
	public Enemy(int pathPattern) {
		this.pathPattern = pathPattern;
	}
	
	public void draw() {
		if (pathPattern == 1) {
			x += 4;
		}
		else if (pathPattern == 2) {
			y +=10;
		}
		else if (pathPattern == 4) {
			x += 4;
			y +=10;
		}
		// ...
	}
}
```
<cap>Enemy 캐릭터의 움직이는 경로를 패턴화하는 코드</cap><br>

만일 Enemy 캐릭터에 새로운 경로 패턴을 추가하고 싶다면,  우리는 매번 새로운 if 블록을 추가하는 식으로 draw() 메서드를 수정해야 한다. 즉, Enemy 클래스는 닫혀있지 않다.

<br>

![image](https://github.com/user-attachments/assets/48f0e1a8-4ad7-4923-a97d-97e311ea53ab)

```java
public class Enemy extends Character {
	private PathPattern pathPattern;
	
	public Enemy(PathPattern pathPattern) {
		this.pathPattern = pathPattern;
	}
	
	public void draw() {
		int x = pathPattern.nextX();
		int y = pathPattern.nextY();
		// ...
	}
}
```

위와 같이 PathPattern을 추상화 하는 것으로, 우리는 Enemy클래스의 변경 없이 새로운 경로 패턴을 추가할 수 있다.

## 3. 리스코프 치환 원칙

> 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.

<br>

리스코프 치환 원칙은 **개방 폐쇄 원칙을 받쳐주는 다형성에 대한 원칙을 제공**한다. 

위 원칙을 간단한 코드로 나타내면 아래와 같다.

```java
public void someMethod(SuperClass sc) {
	sc.someMethod();
}

//...

someMethod(new SubClass()); //정상 동작 해야한다.
```
<cap>상위 타입의 객체를 사용하는 메서드</cap><br>

### 리스코프 치환 원칙의 위반

리스코프 치환 원칙은 기능의 명세에 대한 내용이다. 하위 타입이 상위 타입의 기능 명세를 위반했을 때 발생한다.

그 사례로는 아래와 같은 내용이 있다.

{: .box-yello}
- 명세에서 벗어난 기능을 수행한다.
- 명세에서 벗어난 값을 리턴한다.
- 명세에서 벗어난 익셉션을 발생한다.


#### 명세에서 벗어난 기능을 수행한다.

```java
public class Rectangle { //직사각형
	private int width;
	private int height;
	
	public void setWidth(int width) {
		this.width = width;
	}
	public void setHeight(int height) {
		this.height = height;
	}
	public int getWidth() {
		return width;
	}
	public int getHeight() {
		return height;
	} 
}
```

```java
public class Square extends Rectangle { //정사각형
	@Override
	public void setWidth(int width) {
		super.setWidth(width);
		super.setHeight(width);
	}
	
	@Override
	public void setHeight(int height) {
		super.setWidth(height);
		super.setHeight(height);
	}
}
```

위 예시는 리스코프 치환 원칙을 설명할때 자주 사용되는 직사각형-정사각형 문제이다.

Rectangle 클래스는 직사각형을, 그리고 Square 클래스는 Rectangle을 상속받아 정사각형을 구현한다.

이때, Rectangle의 height를 수정하는 메서드를 살펴보자.

<br>

```java
public void increaseHeight(Rectangle rec) {
	if (rec.getHeight() <= rec.getWidth()) {
		rec.setHeight(rec.getWidth() + 10);
	}
}
```
<cap>Rectangle의 높이를 조절하는 메서드</cap><br>

위 메서드는 직사각형의 높이를 10 높이는 의도를 가지고 있다. 따라서 개발자들은 increaseHeight()의 실행 이후 직사각형은 **서로 다른 width와 height를 지니고 있다고 예상할 것**이다.

그러나 increaseHeight의 인자로 **Square 객제가 전달되는 경우 위 가정은 깨지게 되며, increaseHeight()의 의도와 다른 방식으로 동작하게 될 것**이다.

<br>

```java
public void increaseHeight(Rectangle rec) {
	if (rec instanceof Square) {
		// ... 예외 발생
	}
	if (rec.getHeight() <= rec.getWidth()) {
		rec.setHeight(rec.getWidth() + 10);
	}
}
```

instanceof를 사용한다면 위 문제가 해결될 수 있으나, 앞서 살펴봤듯 Instanceof의 사용은 .그 자체가 개방 폐쇄 원칙을 위반한다.

위 문제 직사각형과 정사각형이 <red>개념적으로는 상속관계처럼 보이나, 실제로는 그렇지 않기에 발생</red>한다. 정사각형은 직사각형의 명세에서 벗어난 기능(가로와 세로폭을 동일하게 고정하는)을 수행한다.

따라서 실제 구현에서는 정사각형과 직사각형을 별개의 타입으로 구현해주어야 한다.

#### 명세에서 벗어난 값을 리턴한다.

```java
public class CopyUtil {
	public static void copy(InputStream is, OutputStream out) {
		byte[] data = new byte[512];
		int len = -1;
		
		// InputStream의 read 메서드는 스트림의 끝에 도달하면 -1을 반환한다. 
		while((len = is.read(data)) != -1) {
			out.write(data, 0, len);
		} 
	}
}
```

CopyUtil은 InputStream의 **read 메서드는 스트림의 끝에 도달하면 -1을 반환한다**는 명세를 사용하여 작성된 코드이다.

이때, InputStream을 상속받는 하위 클래스 SatanInputStream을 아래와 같이 구현했다.

<br>

```java
public class SatanInputStream extends InputStream {
	public int read(byte[] data) {
		return 0; //스트림의 끝에 도달하면 0을 반환
	}
}
```

만일 개발자가 CopyUtil의 copy 메서드에 `InputStream` 대신 `SatanInputStream`을 전달한다면, `is.read(data)`는 스트림의 끝에서 0을 반환하기에 무한루프를 발생시킨다.

이는 **하위 클래스 SatanInputStream가 상위 클래스 InputStream의 명세에서 벗어난 값을 리턴했기에 발생하는 문제**이다. 

## 4. 인터페이스 분리 원칙

> 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다. 
(클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다)

<br>

인터페이스 분리 원칙은 C나 C++과 같이 컴파일과 링크를 직접 해주는 언어를 사용할때 장점이 잘 드러난다.

그러나 C와 C++에서의 설명은 다른 책이나 포스팅을 찾아보도록 하고, 이 포스팅에서는 Java를 사용하여 설명하도록 하겠다… (현재 공부하고 있는 책에서는 C로 예시를 들었다.) 

<br>

```java
public interface Worker {
    void work();
    void eat();
}

// 사무직 직원
public class OfficeWorker implements Worker {
    @Override
    public void work() {
        // ...
    }

    @Override
    public void eat() {
        // ...
    }
}

// 로봇
public class Robot implements Worker {
    @Override
    public void work() {
        // ...
    }

    @Override
    public void eat() {
        // 해당 기능이 필요하지 않으나, 인터페이스에 의해 구현해만 한다!
    }
}

```

위와 코드는 `Worker` 인터페이스를 사용하는 `OfficeWorker`와 `Robot`의 예시이다. 

Robot은 `eat()` 메서드가 필요하지 않으나, Worker 인터페이스를 상속받은 이상 `eat()` 메서드를 구현해야한다.

또한 Worker 메서드에 새로운 메서드를 추가한다면, 해당 **인터페이스를 사용하는 클래스 모두에서 변경이 발생**하게 된다. 해당 메서드가 필요하지 않더라도 말이다.

<br>

```java
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public class OfficeWorker implements Workable, Eatable {
    @Override
    public void work() {
        // ...
    }

    @Override
    public void eat() {
        // ...
    }
}

public class Robot implements Workable {
    @Override
    public void work() {
        // ...
    }
}

```
<cap>인터페이스 분리 원칙이 적용된 코드</cap><br>


Worker 인터페이스를 각 클라이언트(OfficeWorker, Robot)이 필요로 하는 인터페이스로 분리한다면 위와 같은 문제는 사라진다.

이렇게 클라이언트 입장에서 사용하는 기능만을 제공하도록 인터페이스의 책임을 분리하는 것은  단일 책임 원칙과도 연결된다. 즉 인터페이스 분리 원칙을 따르는 것으로 **인터페이스와 콘크리트 클래스의 재사용성을 높일 수 있다.**

## 5. 의존 역전 원칙

> 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다. 
저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 한다.
> 

<br>

![image](https://github.com/user-attachments/assets/8ccdb27b-523c-4a5c-8bdb-f5c88e65cb98)

고수준 모듈과 저수준 모듈은 위와 같이 구분할 수 있다.

즉, 저수준 모듈은 고수준 모듈에서 사용하는 하위 기능을 실제로 어떻게 구현하는가에 대한 내용을 담고있다.

### 고수준 모듈의 저수준 모듈 의존

프로젝트의 초기 요구사항이 안정화된 이후에는, 큰 틀보다는 상세 수준에서의 변경이 주로 발생한다. 따라서 만일 고수준 모듈이 저수준 모듈을 의존하고 있다면, 저수준 모듈의 변경에 따라 고수준 모듈도 함께 변경되게 된다.

```java
public int caculate() {
	// ...
	
	if (someCondition) {
		CouponType1 type1 = ...
		...
	}
	else {
		CouponType2 type2 = ...
		...
	}
	// Coupon이란 저수준 모듈이 추가될때마다
	// 고수준 모듈인 가격 계산 모듈이 변경된다.
}
```

위는 쿠폰의 종류에 따라 가격을 할인해주는 가격 계산 모듈의 예시이다.  이때, 가격 계산 모듈은 개별적인 쿠폰 구현에 의존하고 있다.

이는 **쿠폰의 구현이 추가되거나 변경될 때마다 가격 계산 모듈의 변경을 야기한다.**

### 저수준 모듈의 고수준 모듈 의존

![image](https://github.com/user-attachments/assets/dfcee392-707f-46f9-a830-9247fa35e2e9)


**추상화**를 사용하면 저수준 모듈이 고수준 모듈을 의존하도록 할 수 있다.

앞서 살펴봤던 예시인 `ReservationOrder`의 예시를 살펴보면, 고수준 모듈인 `DashboardHelper`와 하위 모듈인 `OnlineReservation`과 `OfflineReservation` 모두 `ReservationOrder`에 의존하는 것으로 고수준 모듈의 변경 없이 저수준 모듈을 변경할 . 수있는 유연함을 얻을 수 있다.

이때 만들어지는 ReservationOrder의 인터페이스는 저수준 모듈보다는 고수준 모듈인 DashboardHelper의 입장에서 만들어진다.

<br>

```java
public interface ReservationOrder {
	Integer getTotalPriceOfPaidOrder();

	Boolean isMenuAndStatusConfirmed();

	Boolean isStatusConfirmedAndPaidWithoutNoShow();

	String getMenuName();
}

```
<cap>DashboardHelper의 입장에서 만들어진 ReservationOrder</cap><br>


이것은 고수준 모듈이 저수준 모듈에 의존했던 상황이 역전되어, **저수준 모듈이 고수준 모듈이 고수준 모듈에 의존하게 된다는 것**을 의미한다.

<br>

{: .box-note}
> 저수준 모듈이 고수준 모듈을 의존하도록 변경되었으나, 이는 소스코드상에서의 이야기이지 런타임에서의 의존은 여전히 고수준 모듈의 객체에서 저수준 모듈의 객체로 향한다.
> 
> 런타임에서의 의존과 소스코드의 의존을 구분하도록 하자.  