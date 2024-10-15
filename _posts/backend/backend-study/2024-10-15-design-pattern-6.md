---
layout: post
title: "디자인패턴(6)- 주요 디자인 패턴 (1/2)"
author: pam
categories: Backend-Study
description: "전략 패턴, 템플릿 메서드 패턴, 상태 패턴, 데코레이터 패턴, 프록시 패턴, 어댑터 패턴에 대해 학습한다."
tags: Java 디자인패턴
image: https://github.com/user-attachments/assets/00b5a7af-132d-43a3-a030-6631c30be9cf
toc: true
---

---

이전 포스팅

- [객체지향](https://lcqff.github.io/design-pattern-1/)
- [다형성과 추상 타입](https://lcqff.github.io/design-pattern-2/)
- [재사용: 상속보단 조립](https://lcqff.github.io/design-pattern-3/)
- [설계 원칙 SOLID](https://lcqff.github.io/design-pattern-4/)
- [DI(Dependency Injection)와 서비스 로케이터](https://lcqff.github.io/design-pattern-5/)

---
# 개요

<div class="callout">
  <div>💡</div>
  <div>
	<strong>디자인 패턴</strong><br/>
	객체지향설계에서 반복적으로 사용되는 설계가 갖는 일정한 패턴
  </div>
</div>

객체지향의 목표는 문제를 다루면서도 동시에 재설계를 최소화 하는데에 있다.

이러한 객체지향설계는 과거에 사용했던 설계를 재사용하는 경우가 발생하고, 이러한 설계는 **클래스, 객체 구성, 메세지 흐름**과 같은 부분에서 **특정한 패턴**을 가지곤한다.

이러한 패턴을 모아 집대성한 책 중에 유명한 것이 GoF(Gang of Four)이다. 

이번 포스팅에서는 GoF에서 다루는 패턴 중 자주 사용되는 패턴 일부와, 추가적으로 Null 객체 패턴에 대해서 정리할 예정이다. 




> **이번 게시글에서 다룰 내용은...**  
> 전략 패턴   
> 템플릿 메서드 패턴  
> 상태 패턴  
> 데코레이터 패턴  
> 프록시 패턴  
> 어댑터 패턴  

---

<br>

# 전략(Strategy) 패턴

> 특정 콘텍스트에서 알고리즘(전략)을 별로도 분리하는 설계 방법
 

```java
public class Calculator {
	public int calculate(boolean firstGuest, List<Item> items) {
		int sum = 0;
		for (Item item : items) {
			if (firstGuest) {
				// ... 첫번째 손님 할인 정책
			} else if (!item.isFresh()) {
				// ... 덜 신선한 음식 할인 정책
			} else {
				sum += item.getPrice();
			}
		}
		return sum;
	}
}
```

위 코드는 매장 상황에 따라 음식의 가격을 할인해주는 클래스이다.
매장에는 ‘첫 손님 할인 정책’과 ‘덜 신선한 과일 할인’의 두가지 할인 정책이 존재한다.

그러나 현재 Calculator 코드는 아래와 같은 문제를 지니고있다

- 정책이 추가될수록 코드가 복잡해진다.
- 정책이 많아질수록 caculator 메서드를 수정하기 어려워진다.

![image](https://github.com/user-attachments/assets/0f649fc4-4bb6-4240-a0af-089c1285b0c2)

전략 패턴을 사용하여 Calculator 클래스로부터 **할인 정책을 별도 객체로 분리**하면 위 문제를 해결할 수 있다.

이때 분리되는 가격 할인 알고리즘의 추상화를 **전략(Strategy)**, 실제 가격 계산 기능의 책임을 가지고 있는 클래스를 **콘텍스트(Context)**라고 부른다.



```java
public class Calculator {
	private DiscountStrategy discountStrategy;
	
	// 의존성 주입
	public Caclulator(DiscountStrategy discountStrategy) {
		this.discountStrategy = discountStrategy;
	}
	
	public int caclulate(List<Item> items) {
		int sum = 0;
		for (Item item : items) {
			sum += discountStrategy.getDiscountPrice(item);
		}
		return sum;
	}
}
```

전략 패턴의 콘텍스트는 사용할 전략을 직접 선택하지 않고, 의존성 주입을 통해 주입받는다.
이로써 콘텍스트와 전략 객체(전략 콘크리트 클래스)는 동일한 클라이언트에서 생성되게 된다.

```java
public class Client {
	private DiscountStrategy strategy;
	
	...
	
	// 첫 손님 할인 버튼
	public void onFirstGurestButtonClick() {
		strategy = new FirstGuestDiscountStrategy();
	}
	
	//계산 버튼
	public void onCalculatorButtonClick() {
		Calculaotor cal = new Calculaotr(strategy);
		int price = cal.calculate(items);
		...
	}
}
```

클라이언트 코드에서 콘텍스트와 전략 객체를 생성하는 것을 확인할 수 있다.
이는 클라이언트가 **전략의 상세 구현(전략 객체) 대한 의존성을 지닌다**는 것을 뜻한다.

그러나 콘텍스트의 클라이언트가 전략의 인터페이스가 아닌 상세 구현(전략 객체)에 의존해도 괜찮은 걸까?

![image](https://github.com/user-attachments/assets/19cadfcd-b5d6-4dec-bd29-0ee12e00382e)
<cap>전략 객체의 상세 구현 대한 의존성을 지니는 클라이언트 클래스</cap><br>



이 경우 클라이언트 코드와 전략의 콘크리트 클래스가 쌍을 이루기에 문제가 발생할 가능성이 줄어든다고 한다.

만일 전략이 추가될 경우 클라이언트 클래스에서도 해당 전략을 처리하기 위한 코드가 생기고, 전략이 삭제될 경우에도 클라이언트 클래스의 코드가 삭제된다.
 이는 오히려 코드 이해를 높이고 코드 응집을 높여주는 효과를 지닌다.

전략 패턴을 사용함으로써 Caculator 클래스는 **정책 확장에는 열려있고 변경에는 닫혀있는 개방 폐쇄 원칙** 구조를 지니게 된다.

# 템플릿 메서드(Template Method) 패턴

> 여러 클래스들의 실행 과정/단계는 동일하고 각 단계 중 일부의 구현이 다른 경우

예를 들어 DB 데이터를 사용하여/LDAP을 사용하여 사용자 인증을 처리하는 클래스가 있다.
이 두 클래스는 사용자 정보를 가져오는 부분의 구현만 다를 뿐 인증을 처리하는 과정은 완전히 동일하다.

![image](https://github.com/user-attachments/assets/a22ca9d7-15bf-4ef0-80ee-62879421ed82)

템플릿 메서드 패턴은 상위 클래스와 하위 클래스, 두 가지로 구성된다.

- 상위 클래스: 실행 과정을 구현
- 하위 클래스: 실행 과정의 일부 단계 구현



## 상위 클래스

상위 클래스는 **실행 과정을 구현한 메서드**를 제공하며 이 메서드들은 **기능 구현에 필요한 각 단계를 정의**한다.

이때, **구현이 다른 일부 단계는 추상 메서드를 호출**하는 방식으로 구현된다.

```java
public abstract Authenticator {

	//템플릿 메서드
	public Auth authenticate(String id, String pw) {
		//사용자 정보로 인증 확인
		if (! doAuthenticate(id, pw)) {
			// 인증 실패 처리
			throw createException();
		}
		//인증 성공 처리
		return createAuth(id);
	}
	
	protected abstract boolean doAuthentication(String id, String pw);
	
	private RuntimeException createException() {
		throw new AuthException();
	}
	
	protected abstract Auth createAuth(String id);
}
```

상위 추상 클래스 `Authenticator`는 DbAuthenticator와 LdapAuthenticator의 **동일한 실행 과정을 구현**하고 있다. 

이때 사용하는 `authenticate()` 메서드는 **모든 하위 타입에 동일하게 적용되는 실행 과정을 제공**하기 때문에, 이 메서드를 <blue>템플릿 메서드(template method)</blue>라고 부른다.

이때 템플릿 메서드의 경우 외부에 제공하는 기능이기 때문에 **public** 범위를, 그 외 추상 메서드들은 템플릿 메서드에서만 호출되는 메서드로서 하위 타입에서 재정의 되기 때문에 **protected** 범위를 지니고 있어야 한다.

## 하위 클래스

```java
public class LdapAuthenticator extends Authenticator {
	@Override
	protected boolean doAuthenticate(String id, String pw) {
		return ldapClient.authenticate(id, pw);
	}
	
	@Override
	protected boolean createAuth(String id) {
		LdapContext ctx = ldapClient.find(id);
		return new Auth(id, ctx.getAttribute("name"));
	}
} 
```

하위 클래스 `LdapAuthenticator`에서는 전체 실행과정이 아닌 상위 클래스의 추상 메서드의 구현만을 제공하게 된다.

템플릿 메서드 패턴을 사용하면 동일한 실행 과정을 제공하되, 하위 타입에서 일부 단계를 구현하는 것으로 **코드 중복을 방지하고 코드를 재사용** 할 수 있다.

```java
public class SuperCar extends ZetEngine {
	@Overried
	public void turnOn() {
		//하위 클래스에서의 흐름 제어
		if (notReady)
			beep();
		else
			super.turnOn();	
	}
}
```

또한 템플릿 메서드는 특징으로 **상위 타입의 템플릿 메서드가 모든 실행 흐름을 제어**한다는 점을 가지고 있다.

보통 하위 타입이 상위 타입의 기능을 재사용할지 여부를 결정하기 때문에, 하위 타입이 흐름제어를 하는 경우가 일반적이다.


## 템플릿 메서드와 전략 패턴의 조합

그러나 ["재사용: 상속보단 조립"](https://lcqff.github.io/design-pattern-3/) 포스트에서 알아봤듯이, 상속을 통한 재사용은 여러가지 단점들을 가지고 있다. 

템플릿 메서드와 전략 패턴을 함께 사용하면 **상속이 아닌 조립의 방식으로 템플릿 메서드 패턴을 활용**할 수 있다.
(템플릿 메서드는 상속에 기반한 패턴으로 정의되기에, 정확히 설명하자면 템플릿 메서드 패턴보다는 전략 패턴에 가깝다.)

대표적인 예시로는 **스프림 프레임워크의 Template 클래스**가 있다.

```java
public <T> T execute(TransactionCallback <T> action) throws TransactionException {
	....
	
	TransactionStatus status = this.transactionManager.getTransaction(this);
	T result;
	
	//시작, 커밋, 롤백의 실행 흐름을 구현하고있다.
	try {
		// TransactionCallback 전략의 구현체인 action을 파라미터로 전달 받아 사용한다.
		result = action.doInTransaction(status);
	}
	catch (RuntimeException ex) {
		rollbackOnException(status, ex);
		throw ex;
	}
	
	...
	
	this.transactionMAnager.commit(status);
	return result;
}
```

 `execute()`은 **템플릿 메서드**로서 트랜젝션의 **시작/커밋/롤백의 실행 흐름을 제공하고 있다.**

일반적인 템플릿 메서드와 달리 하위 타입에서 재정의될 메서드를 호출하는 것이 아닌, 파라미터로 전달받은 action의 메서드를 호출하고 있다.

이때 action의 타입인 `TransactionCallback`은 함수형 인터페이스로, **action은 `TransactionCallback` 전략의 전략 객체로 동작**한다.

```java
public class MyService {
    private final TransactionTemplate transactionTemplate;

    public MyService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public void executeBusinessLogic() {
        transactionTemplate.execute(status -> {
            // TransactionCallback 인터페이스의 구현 작성
            return null;
        });
    }
}
```

템플릿 메서드 `execute()`을 사용할 때는 위와 같이 파라미터로 transactionTemplate의 구현체를 전달하는 것으로 사용할 수 있다.

# 상태(state) 패턴

> 상태에 따라 동일한 기능 요청의 처리에 차이가 있는 경우 사용하는 패턴

```java
public class VendingMAchine {
	public static enum State {NOCOIN, SELECTABLE}
	
	private State state = State.NOCOIN;
	
	public void insertCoin(int coin) {
		switch(state) {
			case NOCOIN:
				increaseCoin(coin);
				state = State.SELECTABLE;
				break;
			case SELECTABLE:
				increaseCoin(coin);
		}
	}
	
	public void select(int productId) {
		switch(state) {
			case NOCOIN:
				break;
			case SELECTABLE:
				provideProduct(productId);
				dereaseCoin();
				if (hasNoCoin) state = State.NOCOIN;
		}
	}
	
	....
}
```

위 코드는 하나의 제품만을 판매하는 자판기를 구현한 클래스이다.\

자판기에는 동전을 넣는 기능과 물건을 선택하는 기능 두가지가 존재하며, 두 기능 모두 **동전의 존재 상태(NOCOIN)와 제품 선택 가능 상태(SELECTABLE)에 따라 다른 코드를 실행**한다.

위 코드에서 만일 새로운 상태(ex. 자판기에 제품이 존재하지 않는다)가 추가될 경우, 개발자는 `insertCoin()`와 `select()` 메서드에 새로운 조건문을 추가해야한다.

상태가 많아질수록 코드는 복잡해지고, 수정은 어려워 진다.

![image](https://github.com/user-attachments/assets/e7e0e866-992d-40cd-85bb-07c9488b1032)

상태 패턴에서는 상태를 별도의 타입으로 분리하고, 상태별로 알맞은 하위 타입을 구현한다.

기존 코드에서는 VendingMachine이 기능을 제공했지만, 상태 패턴에서는 상태 객체가 기능을 제공한다.

- **상태**: 모든 상태에 동일하게 적용되는 기능 정의
- **콘텍스트**: 기능 실행 요청을 받으면 상태 객체에 처리를 위임

```java
public class VendingMachine {
	private State state;
	
	public VendingMachine() {
		state = new NoCoinState();
	}
	
	public void insertCoin(int coin) {
		state.increaseCoin(coin, this) //위임
	}
	
	public void select (int productId) {
		state.select(productId, this); //위임
	}
	
	public void changeState(State newState) {
		this.state = newState;
	}
	
	....
}
```

```java
public class NoCoinState implements State {

	@Override
	public void increaseCoin(int coin, VendingMachine vm) {
		vm.increaseCoin(coin);
		vm.changeState(new SelectableState());
	}
	
	@Override
	public void select(int productId, VendingMachine vm) {
		SoundUtil.beep();
	}
}
```

이렇게  상태 패턴을 적용하는 것으로 기존 콘텍스트에 구현되어있던 상태별 동작 구현 코드가 각 상태 객체로 이동함을 알 수 있다.

이로서 새로운 상태가 추가되더라도 콘텍스트 코드가 받는 영향은 최소화되고, 상태별 동작의 수정이 필요할 때도 쉽게 수정할 수 있다.

## 상태 변경

```java
public class NoCoinState implements State {

	@Override
	public void increaseCoin(int coin, VendingMachine vm) {
		vm.increaseCoin(coin);
		//콘텍스트의 상태 변경
		vm.changeState(new SelectableState());
	}
	
	@Override
	public void select(int productId, VendingMachine vm) {
		SoundUtil.beep();
	}
}
```

상태 변경의 주체는 콘텍스트와 상태 객체 둘 중 하나가 된다. 위 예제에서는 상태 객체에서 콘텍스트의 상태를 변경하였다.

콘텍스트이 상태 변경을 누가 할지는 주어진 상황에 따라 알맞게 정해주어야 한다.

- 콘텍스트에서 상태를 변경하는 경우
    - 상태 종류가 지속적으로 변경되거나 상태 변경 규칙이 자주 바뀌는 경우 코드가 복잡해지고 상태 변경의 유연함이 떨어진다.
- 상태 객체에서 콘텍스트 상태를 변경하는 경우
    - 상태 변경 규칙이 여러 클래스에 분산됨
    - 한 상태 클래스에서 다른 상태 클래스에 대한 의존 발생

# 데코레이터(Decorator) 패턴

> 위임을 하는 방식으로 기능을 확장한다.

상속은 쉽게 기능의 확장을 가능하게 하는 대신, 다양한 조합의 기능 확장이 요구되는 경우 클래스가 불필요하게 증가한다는 단점을 가지고 있다.

![image](https://github.com/user-attachments/assets/61042e8c-6aab-41af-926f-4a45f4c06c1b)

예를 들어 데이터를 출력하는 기능인 FileOut도, 버퍼 기능을 추가하느냐, 압축 기능을 추가하느냐, 암호화 기능을 추가하느냐의 조합에 따라 위와 같이 클래스와 계층 구조가 복잡해진다.

![image](https://github.com/user-attachments/assets/ff92b1cb-9d79-4c76-8801-92a63fc525da)

데코레이터 패턴을 적용한 구조

데코레이터 패턴은 상속이 아닌 위임의 형태로 기능을 확장한다.

실제 파일 출력 기능의 구현을 위해 `FileOutImpl` 클래스를 만들고, **기능 확장**을 위해선 **Decorator 추상 클래스**를 만들었다.

```java
public abstract class Decorator implements FileOut {
	private FileOut delegate;
	
	public Decorator(FileOut delegate) {
		this.delegate = delegate;
	}
	
	protected void doDelegate(byte[] data) {
	 //delegate에 쓰기 위임
		delegate.wrtie(data);
	}
}
```

`Decorator` 클래스는 모든 데코레이터를 위한 기본 기능을 제공하는 추상 클래스이다.

의존성 주입으로 전달받은 `FileOut` 객체에 `write()` 기능을 위임하고 있다.

```java
public abstract class EncrptionOut implements Decorator {
	
	public EncrptionOut(FileOut delegate) {
		super(delegate);
	}
	
	protected void write(byte[] data) {
	 byte [] encryptedData = encrypt(data);
	 super.doDelegate(encryptedData);
	}
	
	...
}
```

```java
FileOut delegate = new FileOutImpl();
FileOut fileOut = new EncrptionOut(delegate);
fileOut.write(data);
```

위와 같이 코드를 구성함으로서, 최종적으로는 encrypt()에 의해 암호화 된 데이터를 `FileOutImpl`의 write() 메서드를 통해 출력할수 있게 된다.

이는 `EncrptionOut`을 통해 `FileOutImpl`의 **기본 파일 쓰기 기능에 암호화 기능을 추가**해 줄 수 있다. 

이렇게 기존 기능에 새로운 기능을 추가해준다는 의미에서 `EncrptionOut`와 같은 객체를 **데코레이터**라 부른다.

데코레이터 기능을 사용하면 데코레이터를 조합하는 방식으로 기능 확장이 가능하다. 

```java
FileOut delegate = new FileOutImpl();
FileOut zippedFile = new ZipOut(delegate);
FileOut encryptZippedFile = new EncrptionOut(zippedFile);
fileOut.write(encryptZippedFile);
```

# 프록시(proxy) 패턴

> 실제 객체를 대신하는 프록시 객체를 사용하여 실제 객체의 생성이나 접근을 제어
> 

여러개의 이미지가 존재하는 웹페이지를 스크롤해 본다고 하자. 

이때 웹페이지에는 다수의 이미지가 존재하지만, 실제로 사용자의 눈에 들어오는 이미지는 스크린 영역에 존재하는 이미지 뿐이다.

웹페이지에 접속시 사용자의 스크린에 보이지 않는 이미지까지 모두 로딩한다면 불필요한 메모리를 낭비하게 되고 이미지 로딩 시간이 길어지게 된다.

이런 문제를 해결하기 위해 필요 시점에 Image 클래스를 이용하여 이미지를 로딩하는 `DynamicLoadingImage` 클래스를 추가할 수 있다.

![image](https://github.com/user-attachments/assets/f97308b5-4993-4583-8b42-61e1e335b1fc)

그러나 이 경우 `ListUI`는 `DynamicLoadingImage`에 의존하기에, 이미지 로딩 방식과 같은 변경이 발생할시 **ListUI가 함께 변경되는 문제**가 발생한다.

![image](https://github.com/user-attachments/assets/e8885fa3-eacd-4350-98ad-25aa717de40a)

프록시 패턴을 사용하면 `ListUI`의 변경 없이 이미지 로딩 방식을 교체할 수 있게 된다.

- RealImage: 실제로 이미지 데이터를 로딩하여 메모리에 보관하는 콘크리트 클래스

```java
public class ProxyImage implements Image {
	private String path;
	private RealImage image;
	
	public ProxyImage(String path) {
		this.path = path;
	}
	public void draw() {
		if (image == null) {
			// 이미지가 로딩되지 않은 경우 실제 이미지 객체를 생성한다.
			image = new RealImage(path); 
		}
		// 이미지가 로딩되어 있는 경우 RealImage 객체에 위임한다.
		image.draw(); 
	}
}
```

```java
public class ListUI {
	private List<Image> images;
	public ListUI(List<Image> images) {
		this.images = images;
	}
	
	public void onScroll(int start, int end) {
	// 스크롤이 되는 시점에 Image 객체의 draw 메서드를 실행시킨다.
		for(int i = start; i <= end; i++) {
			Image image = images.get(i);
			image.draw();
		}
	}
}
```

ListUI 객체는 스크롤 되는 시점에 Image 타입의 draw 메서드를 호출한다. 

만일 ListUI가 지닌 Image가 `ProxyImage`인 경우, 이미지가 로딩되어있지 않는 경우 `RealImage`을 생성하고, 이미지가 로딩되어 있는 경우 `RealImage` 객체에 위임한다. 

만일 상위 이미지 4개는 미리 로딩해둬야 하는 경우 ListUI가 지니고 있는 images 필드의 가장 처음 4개 객체는 RealImage를, 나머지는 ProxyImage를 전달하는 식으로 RealImage객체와 ProxyImage객체를 섞어 사용할 수 있다. 

# 어댑터(Adapter) 패턴

> 클라이언트가 요구하는 인터페이스와 재사용하려는 모듈이 제공하는 인터페이스가 일치하지 않을때 사용하는 패턴

웹 게시판에 통합 검색 기능을 추가하려 한다. 설계는 아래와 같다.

![image](https://github.com/user-attachments/assets/0cee9e51-371f-490e-ad01-2a0df04ad4d7)

그러나 게시글 개수의 증가로 SQL을 이용한 검색 속도 성능에 문제가 발생했다. 

따라서 개발자는 성능 문제를 해결하기 위해 Tolr라는 오픈 소스 검색 서버를 도입하려 한다. Tolr는 `TolrClient`라는 모듈을 제공하기 때문에 쉽게 연동이 가능했다.

![image](https://github.com/user-attachments/assets/348107f5-a50c-4555-9d16-bde38ee7fe6a)

그러나 문제가 발생했다. `TolrClient`의 인터페이스가 `SearchService`의 인터페이스와 맞지 않는다는 점이다.

`SearchService` 대신에 `TolrClient`를 사용하기에는 이미 다른 클래스들은 `SearchService`을 사용하도록 작성되어 있고, `TolrClient`로의 변경은 많은 수정을 요구한다.

이렇게 **클라이언트가 요구하는 인터페이스(SearchService)와 재사용하려는 모듈의 인터페이스(TolrClient)가 일치하지 않을 때** 어댑터 패턴을 사용할 수 있다.

![image](https://github.com/user-attachments/assets/00b5a7af-132d-43a3-a030-6631c30be9cf)

어댑터인 `SearchServiceTolrAdapeter`는 **TolrClient를 SearchService 인터페이스에 맞춰주는 책임**을 지닌다. 

```java
public class SearchServiceTolrAdapeter implements SearchService {
	private TolrClient tolrClient = new TolrClient();
	
	public SearchResult(String keyword) {
		//keyword를 tolrClient가 요구하는 형태로 변환
		TolrQuery tolrQuery = new TolrQuery(keyword);
	
		//TolrClient 기능 실행
		QueryResponse response = tolrClient.query(tolrQuery);
		
		//TolrClient의 결과를 SearchResult로 변환
		SearchResult result = convertToResult(response);
		return result;
	}
	
	....
}
```

`SearchServiceTolrAdapter` 클래스의 `Search()` 메서드는 클라이언트가 제공하는 keyword 파라미터를 **Tolr가 요구하는 TolrQuery로 변환**하고, TolrClient의 결과값을 **클라이언트가 요구하는 SearchResult로 변환**하는 역할을 한다.