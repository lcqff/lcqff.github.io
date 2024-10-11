---
layout: post
title: "애플리케이션 아키텍쳐"
author: 파녀미
categories: Backend-Study
tags: spring
---


## 애플리케이션 아키텍쳐

<span><blue>계층형 아키텍쳐(Layered Architecture)</blue></span>는 각 구성 요소들의 **관심사를 분리**하기 위해 각 계층의 책임을 분리한 아키텍쳐이다. 각 계층의 책임을 분리함으로 각 계층은 하나의 관심사만을 가지게 되고 이로써 계층의 응집도를 높이고 결합도를 낮출 수 있다.  

![계층형 아키텍쳐의 4계층](https://github.com/lcqff/lcqff.github.io/assets/71930280/14021ecd-c510-48b1-a4d6-39963fe4bb51)
<br/><br/>
소프트웨어 개발에 있어 계층은 3 혹은 4계층으로 나누어 진다. 각 계층은 다음과 같다.


1. **Presentation Layer**
    - 가장 바깥 계층으로 사용자와 직접 상호작용하는 계층  
2. **Businness Layer**
    - 요청에 따른 비즈니스 로직을 다루는 계층
3. **Persistence Layer**
    - 데이터베이스와 상호작용하는 계층
4. **Database Layer**
    - 데이터가 저장되어 있는 계층 (데이터베이스)
{: .box-yello}


<red>각 계층은 하위 계층에 의존적이며, 상위 계층을 알지 못해야 한다.</red>
<br/><br/>
<img alt="걔층형 아키텍쳐" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/bb245470-d783-4fea-8029-b0888dafce12">

위와 같은 애플리케이션 구조도를 예시로 들어보자. 이때  
**Controller는** `Presentation Layer`,  
**Service는** `Business Layer`,  
**Repository는** `Persistence Layer`  
에 속하게 된다.  

이때 **Repository는** 데이터베이스와 상호작용하는 계층이므로, JPA와 엔티티 매니저를 직접 사용한다.  



 