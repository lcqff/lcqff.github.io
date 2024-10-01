---
layout: post
title: 디자인패턴(3)- 재사용 | 상속보단 조립
author: 팜
categories: Java
description: \[Study] 상속이 아닌 조립(Composition)을 통해 코드를 재사용하자.
tags: Java 디자인패턴
sidebar:
---

이전 포스팅
- [객체지향](https://lcqff.github.io/java/2024/08/19/design-pattern-1.html)  
- [다형성과 추상 타입](https://lcqff.github.io/java/2024/09/24/design-pattern-2.html)  

---

## 0. 개요

객체 지향의 주요 특징 중 하나는 코드의 재사용이다. 이런 재사용을 위해 사용할 수 있는 여러 방법이 있는데, 나는 주로 상속을 사용해왔다.

그러나 코드를 재사용하기 위해 상속을 사용하는 것은 몇가지 문제점을 초래한다. 이번 챕터에서는 이에 대해 자세히 알아보도록 한다.

## 1. 상속과 재사용

### 상속

Spring MVC의  Controller 계층구조를 통하여 상속에 대해 알아보자.  

![image](https://github.com/user-attachments/assets/80db791a-20d0-4653-9849-f7fb3a369f45)

Spring MVC의 Controller간 상속관계는 위와 같다. 

이때 각 객체는, 자신이 상속받고 있는 상위객체의 기능에 더불어 자신만의 기능을 추가적으로 제공한다. 

이렇게 상속을 상요하면 다른 클래스의 기능을 쉽게 재사용하면서, 추가 기능을 확장할 수 있게된다.

그러나 상속은 <red>변경의 유연성을 해한다는 치명적인 단점</red>을 지니고있다.

## 2. 상속을 통한 재사용의 단점

### 상위 클래스 

![image](https://github.com/user-attachments/assets/5f8b273a-3739-47e2-87ba-5f1166a881f3)

첫번째 포스팅에서 **의존의 전이**에 대해 다뤘었다.


<div class="callout">
  <div>💡</div>
  <div>
    <strong>의존(Dependency)</strong><br/>
    한 객체가 다른 객체를 생성하거나, 파라미터로 전달받거나, 다른 객체의 메서드를 호출하는 것.<br>
	한 객체의 변경사항이 다른 객체의 변경을 줄 가능성이 높아지는 것.
  </div>
</div>


상속또한 다른 객체에 대한 의존을 야기한다. 따라서 의존하는 클래스의 코드의 변경은 해당 클래스를 상속받는 클래스에 또한 변경이 발생할 수 있다.

<br>

![image](https://github.com/user-attachments/assets/a601274b-4b77-4400-a5eb-6a875b6e5d90)

이를태면 `AbstractController`의 변경은 AbstractController을 상속받은 `BaseCommandController`와 `AbstractUrlViewController`의 변경을 불러일으킬수 있다.


또한 이렇게 발생한 `BaseCommandController` 의 변경은 BaseCommandController을 상속받은 `AbstractCommandController`과 `AbstractFormController`의 변경을 불러일으킬 수 있다.

이렇게 상속에 의해 발생한 의존에 의해 변경의 여파는 계층도를 따라 하위 클래스에 계속 전파된다. 

따라서 계층도가 커질수록 상위 클래스의 변경은 점점 어려워진다.

### 클래스의 불필요한 증가

경우에 따라 상속은 기능 확장에 있어서도 쓰임이 불편할 수 있다.

아래의 경우를 예시로 들어보자.

![image](https://github.com/user-attachments/assets/f7bbfa07-deaa-40b6-80a7-6b7476b5c404)

위 클래스 계층도는 파일 보관소를 구현한 `Storage` 클래스와, 그에 압축 기능과 암호화 기능을 추가로 구현한 파일 저장소인 `CompressedStorage`와 `EncryptedStorage`이다.

이제 여기서 기능을 더 확장하려고 한다. 압축을 한 뒤 암호화를 하는 저장소와, 암호화 한 뒤 압축하는 저장소와 암호화된 저장소에 캐시를 적용하는 저장소이다. 상속을 통해 기능을 적용해보자.

<br>

![image](https://github.com/user-attachments/assets/2b952caf-05ed-41a2-afea-d92cfffe8805)

결과적인 클래스 계층은 위와 같아진다.

거기에 만일 Java를 사용한다면, 다중상속이 불가능하기에 한개의 클래스만 상속받고 별도로 기능을 구현해주어야 할 것이다.

이렇게 필요한 기능의 조합이 증가할수록 상속을 통한 기능 재사용은 클래스의 개수 증가를 불러일으킨다.

### 상속의 오용

상속 자체를 잘못 사용하게 되는 경우또한 발생할 수 있다. 이번에도 예시를 통해 알아보자.

```java
public class Container extends ArrayList<Luggage> {
	private int maxSize;
	private int currnetSize;
	
	public void put(Luggage lug) throws NotEnoughSpaceException {
		if(!canConatain(lug)) throw new NotEnoughSpaceException();
		super.add(lug);
		currentSize += lug.size();
	}
}

...
```

`ArrayList`를 상속받는 클래스 `Container`가 있다.

`Container` 클래스는 `put()` 메서드를 통해 현재 Container 내부의 크기를 검증하고, 만일 크기가 초과되지 않는다면 `ArrayList`의 `add()` 메서드를 통해 `ArrayList`에 수화물을 추가한 후 Container의 크기를 증가시킨다.

그러나 위 코드는 아래와 같이 잘못 사용될 수 있다.

<br>

```java
Container c = new Container(5);
if (c.canContain(size3Lug)) {  
	c.put(size3Lug); //정상 사용, currnetSize: 3
}

if (c.canContain(size2Lug)) {
	c.add(size2Lug) //비정상 사용, currnetSize 늘어나지 않음.
}

if (c.canContain(size1Lug)) {
	c.put(size1Lug); // 사이즈 5짜리 컨테이너에 6개의 수화물이 들어갔다!
}
```

Container 클래스를 개발한 개발자가 다른 개발자들에게 수화물을 추가할때  `ArrayList`의 `add()` 메서드가 아닌 `Container`의 `put()` 메서드를 사용하라 일렀을지라도, 다른 개발자들은 IDE의 자동완성이 제시한 `add()` 메서드를 사용할 수 있다.

<br>

위와 같은 실수가 발생한 이유는 `Container`와 `ArrayList` 사이에 <red>IS-A 관계가 성립하지 않기 때문</red>이다.  즉 **Container는 ArrayList가 아니다.**

`Container`는 수화물을 보관하는 책임을 지는 반면 `ArrayList`은 목록을 관리하는 책임을 지닌다. 완전히 다른 책임을 지니는 두 클래스를 상속하 문제가 발생하게 된다. 

<br>

그렇다면 IS-A관계에 해당하지 않는 클래스의 코드를 재사용하기 위해선 무엇을 사용해야하는가? 

## 3. 조립을 이용한 재사용

**객체 조립(Composition)**은 여러 객체를 묶어 더 복잡한 기능을 제공하는 객체를 만드는 기술이다.

객체지향 언어에서 객체 조립은 필드에서 다른 객체를 참조하는 방식으로 구현된다.

```java
public class FlowController {
	private Encryptor encryptor = new Encryptor();
	
	public void process() {
		...
		byte[] encrtptedData = encryptor.encrypt(data);
	}
}
```
<cap>객체 참조의 예시</cap><br>

### 클래스 비증식

앞서 클래스 증식 문제를 야기한 Storage 예제를 다시 살펴보자. 

![image](https://github.com/user-attachments/assets/2b952caf-05ed-41a2-afea-d92cfffe8805)

위와 동일한 기능을 구현하는 객체를 상속이 아닌 객체 조립의 형태로 재구성한다면 아래와 같아진다.

<br>

![image](https://github.com/user-attachments/assets/8d8075d6-a9fa-49ab-be0c-936bdcf51298)

상속 대신 조립을 사용한 객체 Storage는 이제 기능을 확장해도 클래스가 증식하지 않는다. 확장된 기능 또한 조립하면 되기 때문이다.

<br>

그렇다면 Storage가 Compressor와 Encryptor을 상속하는 방식으로 구현할수도 있지 않을까?

그러나 그러한 방식은 Storage 클래스를 저장소 자체가 아닌 압축이나 암호화 목적으로 사용할 수 있게 되며, 경우에 따라 Storage 클래스의 내부 상태가 비정상적으로 변경될 수 있다.

<br>

또한 상속의 경우 런타임에서 상위 클래스를 교체할 수 없으나, 조립을 사용하면 런타임시 조립 대상 객체를 교체할 수 있게된다는 장점또한 존재한다.

### 위임

**위임(delegation)**을 통해 내가 할 일을 다른 객체에게 넘길 수 있다. 이는 보통 조립을 사용하여 구현된다.

아래의 예시를 보자

```java
public abstract class Figure {
	private Bounds bounds = new Bounds();
	
	// ...
	
	private boolean contains(Point point) {
		return bounds.contains(point.getx(), point.getY());
	}
}

```

 위 코드는 마우스 포인터의 위치가 특정 도형이 차지하는 영역에 포함되어 있는지 확인하는 기능을 구현한 코드이다.

도형과 관련된 클래스 Bounds 클래스가 이미 해당 기능을 제공하고 있기 때문에, 도형을 표현하는 클래스인 Figure 클래스는 Bounds 객체에게 포함 여부 확인을 위임하고 있는 걸 확인할 수 있다.

### 상속은 언제 사용하는가?

지금까지 상속보다는 조립을 사용한 코드 재사용의 장점을 알아보았다. 

그렇다면 상속은 언제 사용해야 할까?

상속을 사용할때에는 재사용의 관점이 아닌 **기능의 확장이라는 관점에서 상속을 적용**해야한다.

또한 **명확한 IS-A 관계가 성립**되어야 한다.

상속을 사용한 클래스 계층은 , 상위 클래스의 기본적인 기능을 그대로 유지하면서 그 기능을 확장해나간다는 특징을 가지고 있다.