---
layout: post
title: 디자인패턴(5)- DI(Dependency Injection)와 서비스 로케이터
author: 팜
categories: Java
description: \[Study]서비스 로케이터의 단점과 이를 대체하는 의존성 주입 방법에 대해 학습.
tags: Java 디자인패턴
sidebar:
---

이전 포스팅
- [객체지향](https://lcqff.github.io/java/2024/08/19/design-pattern-1.html)  
- [다형성과 추상 타입](https://lcqff.github.io/java/2024/09/24/design-pattern-2.html)  
- [재사용: 상속보단 조립](https://lcqff.github.io/java/2024/09/30/design-pattern-3.html)
- [설계 원칙 SOLID](https://lcqff.github.io/java/2024/10/01/design-pattern-4.html)

---
## 0. 개요

소프트웨어는 두 개의 영역으로 구분할수 있다. 

- **어플리케이션 영역**: 고수준 정책 및 저수준 구현을 포함
- **메인 영역**: 어플리케이션이 동작하도록 각 객체들을 연결해주는 영역

이때 DI(dependency injection: 의존성 주읩)은 메인영역에서 객체를 연결하기 위해 사용된다. 

<br>

이번 포스팅에서는…

> 어플리케이션 영역과 메인 영역의 차이를 학습한다.
> 
> 객체 연결 방법인 서비스 로케이터와 의존성 주입(DI)에 대해 학습한다.
> 
> 서비스 로케이터의 단점에 대해 학습한다.
> 
> 의존성 주입 방법에 대해 학습한다.

## 1. 어플리케이션 영역과 메인 영역

이번 포스팅에선 학습을 위해 간단한 `비디오 포맷 변환기`예시코드를 작성한다. 
요구사항은 아래와 같다. 

{: .box-note}
- 변환 작업을 요청하면 순차적으로 변환을 처리한다.
    - 변환 요청 정보는 **파일** 또는 **DB**를 이용하여 보관할 수 있다.
- 비디오의 변환 처리는 오픈 소스인 **ffmpeg**를 사용하거나 구매 예정인 **변환 솔루션**을 사용할 수 있다.
- 명령행에서 변환할 source 파일과 변환 결과인 targer 파일을 입력한다.

### 어플리케이션 영역

위 요구사항을 설계로 옮기면 아래 이미지와 같다.

![image](https://github.com/user-attachments/assets/2f600066-2924-4656-902a-e48bef8231f3)

[개방 폐쇄 원칙](https://lcqff.github.io/java/2024/10/01/design-pattern-4.html)을 지키기 위하여 작업을 처리하는 기능과 비디오 변환 기능을 Interface로 구현하였다. 
이로써 작업 처리 방식과 비디오 변환 방식이 추가되어도 transcoder 패키지의 변경은 발생하지 않을 것이다.

`JobQueue`와 `Transcoder`을 사용하여 실제 기능을 구현한 `Worker` 클래스는 아래와 같다.

```java
public class Worker {

	public void run() {
		JobQueue jobQueue = ... //JobQueue을 구현한 콘크리트 객체를 구한다.
		Transcoder transcoder = ... // Transcoder을 구현한 콘크리트 객체를 구한다.
		
		while ( ... ) {
			JobData jobdata = jobQueue.getJob();
			transcoder.transcode(jobData.getSource(), jobData.getTarget());
		}
	
}
```

이때 worker가 `FileJobQueue`와 `DBJobQueue`와 같은 콘크리트 객체를 의존하는 경우, <red>순환의존의 문제</red>가 발생한다.

transcoder 패키지를 의존하고 있는 다른 패키지를 의존하게 되기 때문이다.

또한 `FileJobQueue`와 `DbJobQueue`는 `JobQueue`를 구현한 **하위 모듈**이기에, 상위 모듈 Worker가 하위모듈을 의존하게 되어 <red>의존 역전 법칙을 위반</red>하게 된다.

<br>

![image](https://github.com/user-attachments/assets/bdecdeec-5dce-418d-be7e-6b229bc469b3)

이러한 두 문제는 **JobQueue구현체를 제공하는 클래스를 transcoder 패키지 안에 포함시키는 방법**으로 해결할 수 있다. 

<br>

![image](https://github.com/user-attachments/assets/95030aa2-c945-4d72-8784-995e8067a0a3)

`Locator`는 Worker에 필요한 **JobQueue와 Transcoder의 구현체를 반환하는 역할**을 한다. 이로써 순환의존 문제와 의존 역전 법칙 위반 문제를 해결할 수 있었다.

실제 구현 코드는 아래와 같다.

<br>

```java
public class Worker {

	public void run() {
		JobQueue jobQueue = Locator.getInstance().getJobQueue();
		Transcoder transcoder = Locator.getInstance().getTanscoder();
		
		while ( ... ) {
			JobData jobdata = jobQueue.getJob();
			transcoder.transcode(jobData.getSource(), jobData.getTarget());
		}
	
}
```

```java
public class Locator {

	private static Locator instance;
	
	public static Locator getInstance() {
		return instance;
	}
	
	public static void init(Locator locator) {
		this.instance = locator
	}
	
	private JobQueue jobqueue;
	private Transcoder transcoder;
	
	public Locator(JobQueue jobQueue, Transcoder transcoder) {
		this.jobQueue = jobqueue;
		this.transcodeer = transcoder;
	}
		
	// ... getter
}
```

이렇게 **사용할 객체를 제공하는 책임을 갖는 객체**를 <blue>서비스 로케이터(Service Locator)</blue>라 한다.


<div class="callout">
  <div>💡</div>
  <div>
	<strong>서비스 로케이터(Service Locator)</strong><br/>
    사용할 객체를 제공하는 책임을 가지는 객체
  </div>
</div>


### 메인영역

어플리케이션 영역에서 우리는 Locator을 사용하여 Worker에 콘크리트 객체를 할당해주었다. 

그렇다면 Locator는 언제 초기화되는가? 
즉, **어플리케이션 영역에서 사용되는 객체의 생성**은 누가 담당하는가? 바로 메인 영역이다.

<br>

메인 영역의 책임은 아래와 같다.

- 어플리케이션 영역에서 사용될 객체를 생성한다.
- 각 객체간의 의존 관계를 설정한다.
- 어플리케이션을 실행한다.

<br>

```java
public class Main{
	public static void main(String [] args) {
		//상위 수준 모듈 transcoder 패키지에서 사용할 하위 모듈 생성	
		JobQueue jobQueue = new FileJobQueue();
		Transcoder transcoder = new Transcoder();
	
		//Locator 초기화
		Locator locator = new Locator(jobQueue, transcoder);
		Locator.init(locator);
		
		//상위 수준 모듈 생성 및 실행
		final Worker worker = new Worker();
		Thread t = new Thread(new Runnable) {
			public void run() {
				workder.run();
			}
		}
	}
}
```

위는 메인 영역에서 사용되는 Main 클래스 코드이다.

transcoder 패키지에서 필요로 하는 **하위 모듈을 생성**하고 **Locator에 이를 할당**해주는 것으로 어플리케이션 영역에서 사용되는 객체의 **의존관계를 설정**해주었다.

![image](https://github.com/user-attachments/assets/ecea04e0-44fb-45c0-b9cf-7a6902186aa3)

이러한 설계를 통해 만일 어플리케이션 영역에서 사용하는 하위 수준의 모듈의 변경은 메인 영역을 수정하는 것으로 해결할 수 있다.

**모든 의존은 메인 영역에서 어플리케이션 영역으로 향한다**. 역방향의 의존은 발생하지 않는다.

즉, 메인 영역의 코드를 변경이 어플리케이션 영역의 코드 변경을 야기하지 않는다.

### 서비스 로케이터의 단점

위 예시에서는 **서비스 로케이터**를 이용하여 각 객체간 의존관계를 설정했다.

그러나 서비스 로케이터를 사용하는 방식은 방식은 몇가지 단점이 존재한다.

<br>

- **서비스 로케이터는 동일 타입의 객체가 다수 필요한 경우 각 객체별로 제공 메서드를 만들어주어야 한다.**

```java
public class Worker {

	public void run() {
		JobQueue fileJobQueue = Locator.getJobQueue1();
		JobQueue dbJobQueue = Locator.getJobQueue2();
		// ...	
}
```

```java
public class Locator {

	// ...
	
	private JobQueue jobQueue1;
	private JobQueue jobQueue2;
	
	public Locator(JobQueue jobQueue1, JobQueue jobQueue2) {
		this.jobQueue1 = jobQueue1;
		this.jobQueue2 = jobQueue2;
	}
		
	// ... getter
}
```

<br>

그렇다면 getJobQueue1, getJobQueue2라는 이름 대신에 getFileJobQueue, getDbJobQueue라는 이름을 쓸 순 없는걸까? 

이를태면 아래와 같이 특정 조건에 따라 fileJobQueue를 사용할수도, dbJobQueue을 사용할수도 있는 Worker 클래스가 있다.

```java
public class Worker {

	public void run() {
		// ...
		
		
		JobQueue jobQueue = someCondition ? 
			Locator.getFileJobQueue() : Locator.getDbJobQueue();
		
		// ...	
}
```

그러나 이러한 사용은 결국 Worker 클래스가 JobQueue 구현 객체에 의존하도록 만든다. 
만일 File이나 DB가 아닌 새로운 기능이 추가되는 경우, Worker 클래스의 변경이 발생하게 된다는 것이다. 
즉 개방 폐쇄 원칙을 위반한다. 

<br>

- **인터페이스 분리 원칙을 위반한다.**

<div class="callout">
  <div>💡</div>
  <div>
	<strong>인터페이스 분리 원칙 (ISP)</strong><br/>
    클라이언트는 자신이 사용하지 않는 인터페이스에 의존하지 않는다.
  </div>
</div>

<br>

```java
public class Locator {

	private static Locator instance;
	
	public static Locator getInstance() {
		return instance;
	}
	
	// ....
}
```

```java
public class Worker {

	public void run() {
		JobQueue jobQueue = Locator.getInstance().getJobQueue();
		Transcoder transcoder = Locator.getInstance().getTanscoder();
		...
		// Worker는 Locator에서ㅓ 제공하는 모든 타입에 접근할수 있게 된다. 
	
}
```

서비스 로케이터를 사용하는 객체는 자신이 필요한 타입 뿐만 아니라 서비스 로케이터가 제공하는 모든 타입을 사용할수 있다. 
이렇게 발생하는 **불필요한 의존**으로 인해 다른 의존 객체에 의해 발생하는 변경에 영향을 받을 수 있다.

<br>

 이러한 단점들 때문에 일반적으로는 서비스 로케이터 대신, 외부에서 객체를 주입해주는 **`DI(Dependency Injection)`**이 사용된다.

## 2. 의존성 주입을 사용한 의존 객체 사용


<div class="callout">
  <div>💡</div>
  <div>
	<strong>의존성 주입(DI; Dependency Injection)</strong><br/>
    필요한 객체를 직접 생성하거나 찾지 않고 외부에서 넣어주는 방식
  </div>
</div>

<br>

DI는 서비스 로케이터의 단점을 보완하기 위한 방법으로 그 구현 자체는 단순하다.

```java
public class Worker {
	private JobQueue jobQueue;
	private Transcoder transcoder;
	
	public Worker(JobQueue jobQueue, Transcodeer transcoder) {
		this.jobQueue = jobQueue;
		this.transcoder =. transcoder;
	}
	
	public void run() {
			...
		}
}
```

```java
public class Main{
	public static void main(String [] args) {
		//상위 수준 모듈 transcoder 패키지에서 사용할 하위 모듈 생성	
		JobQueue jobQueue = new FileJobQueue();
		Transcoder transcoder = new Transcoder();
		
		//상위 수준 모듈 생성 및 실행
		final Worker worker = new Worker(jobQueue, transcoder);
		
		...
}
```

worker 객체는 스스로 의존하는 객체를 찾거나 생성하는 과정 없이 **외부의 main 메서드에서 생성자를 통해 사용할 객체를 주입**받는다.

이렇게 외부의 누군가에게 의존할 객체를 주입받는 것을 **의존성 주입**이라 한다. 

### 조립기

DI를 사용할시 각 객체들을 생성하고, 의존 관계에 따라 연결해주는 **조립기**가 필요하다.

위 예제에서는 Main이 객체의 실행과 더불어 조립기의 역할을 하고 있지만 조립기를 별도로 분리하면 조립기 구현 변경의 유연성을 얻을 수 있다.

```java
public class Assembler {

	public void createAndWrite(() {
		JobQueue jobQeue = new FileJobQueue();
		Transcoder transcoder = new FfmpegTranscoder();
		this.worker = new Worker(jobQeue,transcoder);
	}
	
	// ...getter
}
```

```java
public class Main{
	public static void main(String [] args) {
		Assember assembler = new Assembler();
		assembler.createAndWire();
		
		final Worker worker = assembler.getWorker();
		
		...
}
```

## 3. 의존성 주입 방식

### 생성자 주입(Constructor Injection)

```java
public class Worker {
	private JobQueue jobQueue;
	private Transcoder transcoder;
	
	public Worker(JobQueue jobQueue, Transcodeer transcoder) {
		this.jobQueue = jobQueue;
		this.transcoder =. transcoder;
	}
	
	public void run() {
			...
		}
}
```

- 생성자를 통해 의존 객체를 전달받는다.
- 객체 생성 시점에서 의존 객체의 오류를 검증할 수 있다.
- 의존 객체가 먼저 생성되어야만 사용할 수있다.
    - 객체의 생성이 객체의 정상 동작을 보장한다.

### 세터 주입(Setter Injection)

```java
public class Worker {
	private JobQueue jobQueue;
	private Transcoder transcoder;
	
	public void setJobQueue(JobQueue jobQueue) {
		this.jobQueue = jobQueue;
	}
	
	...
}
```

- 객체가 완전히 초기화되지 않은 상태로 사용될 수 있다.
    - 객체의 생성이 객체의 정상 동작을 보장할 수 없다.

### 필드 주입(Field Injection)

```java
public class Worker {
	@Autowired
	private JobQueue jobQueue;
	@Autowired
	private Transcoder transcoder;
	
	...
}
```

- 프레임워크에서 주로 사용된다.
- 코드가 간결해지나 의존성 주입이 명확하게 드러나지 않는다
- 테스트 시 의존성을 설정하기가 어렵다.

## 4. DI와 테스트

![image](https://github.com/user-attachments/assets/55225b75-9aa5-4059-8bfb-a2bb1c45a471)

위와 같은 구조에서, `Worker`를 테스트하기 위해서는 JobQueue의 구현체와 Transcoder의 구현체가 필요하다. 

JobQueue와 Transcoder의 구현체가 완성돼있지 않은 상태에서, `Worker`가 **의존성 주입**을 사용하고 있다면 단순히 Workder에 JobQueue 구현체의 Mock 객체와 Transcoder의 **Mock 객체**를 전달하는 것으로 해결할 수 있다.

이렇게 의존성 주입은 테스트의 편의성을 제공하기도 한다.

## 5. 스프링에서의 의존성 주입

의존성 주입(DI)은 스프링이 제공하는 핵심 기능 중 하나이다.

**스프링 프레임워크**와 **스프링 부트**는 모두 DI를 지원하며, 스프링의 핵심 컨테이너인 **IoC(Inversion of Control)** 컨테이너를 통해 DI를 구현한다.

```java
@Configuration
public class TranscoderConfig {

	@Bean
	public JobQueue fileJobQueue() {
		return new FileJobQueue();
	}
	
	@Bean
	public Transcoder ffmpegTranscoder() {
		return new FfmpegTranscoder();
	}
	
	@Bean
	public Workder worker() {
		return new Worker(fileJobQUeue(),ffmpegTranscoder());
	}
}
```

위는 스프링 프레임워크를 사용한 의존성 주입의 예시이다.

- 개발자가 모든 Bean과 그 관계를 직접 정의한다.
- 외부 종속성의 경우 DI 설정파일에서 의존성을 주입해야한다.

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**스프링 부트**는 스프링 프레임워크의 설정을 간소화하고, 빠르게 애플리케이션을 구축할 수 있도록 돕는 도구로, 의존성 설정 또한 **자동으로 설정 및 관리**된다.

위 코드는 스프링 부트를 사용한 의존성 주입의 예시이다.

- 별도의 Bean 설정이 필요하지 않는다.
- `@SpringBootApplication` 어노테이션을 사용하여 프로젝트 내의 모든 빈과 외부 종속성을 자동으로 탐색 및 등록한다.
- `spring-boot-starter-web`과 같은 스타터 패키지를 제공하여 해당 패키지 사용시 자동으로 MVC 관련 빈들이 등록된다.
- `@SpringBootTest` 어노테이션을 사용하면 테스트시에도 자동으로 DI를 처리해준다.