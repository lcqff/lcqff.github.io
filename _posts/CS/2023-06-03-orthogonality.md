---
layout: post
title: "프로그래밍에서의 직교성(Orthogonality)"
author: 파녀미
categories: CS
banner:
  image: https://github.com/lcqff/lcqff.github.io/assets/71930280/c75af5fd-2b35-49ad-96cd-051081e88588
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: Orthogonality
sidebar: []
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

<div class="captionedImg">
<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/c75af5fd-2b35-49ad-96cd-051081e88588">
<p>https://www.baeldung.com/cs/orthogonality-cs-programming-languages-software-databases</p>
</div>

<div class="callout">
  <div>💡</div>
  <div>
    <strong>직교성(Orthogonality)</strong><br/>
    프로그래밍 언어의 다양한 기능들을 모든 방법으로 조합 가능하게 만드는 것
  </div>
</div>
- 기능의 독립성이 보장되어 있다. 
- 임의의 여러 기능이 조합 가능하다.

---

C는 언어가 가진 여러 특징 때문에 직교성이 낮은 언어로 알려져 있습니다.
하지만 이는 C가 직교성을 가지고 있지 않다는 뜻은 아니며, C 또한 여러가지 직교성을 제공합니다.

C가 제공하는 직교성의 예시와, C에서 발생하는 직교성에 기반한 오류의 예시를 살펴보며 직교성에 대해 알아봅시다.

#### C가 제공하는 Orthogonality와 그에 기반된 오류

```c
int x = 1;
float x = 2.0;
```

위는 변수 x에 대해 자료형과 값에 대한 변경이 이루어지는 예시입니다. 이와 같이 C에서는 변수의 `자료형 지정`과 `변수명 선언`, `변수 초기화`에 대해 기능의 독립성이 보장되있음을 알 수 있습니다.

##### 직교성에 의한 오류

<br/>
<br/>

```c
int x = 1;
float y = 2.15;
int z = x + y;
```

위의 코드는 프로그래머의 실수로 정수형으로 선언된 변수 z에 기존의 정수형 변수와 실수형 변수 x, y의 합이 할당되는 코드입니다.
z는 정수형으로 선언되어 있기에 직교성으로 인해 실수형 변수인 y에 대해 자동으로 형변환이 이루어집니다. 이는 프로그래머의 의도와 다르게 y의 소수점 아래의 값을 유실시키는 결과를 낳습니다.

<br/><br/><br/>

---

```c
if (x = y)
```

위는 if의 조건문에 `x = y`라는 할당문이 입력되어있는 예시입니다. C는 조건문 안에 할당문을 삽입하는 것을 허용하며, 따라서 위 코드는 오류 없이 실행 가능합니다.
위 코드는 조건문과 할당문의 조합으로 이루어져 있으며 오류 없이 동작 가능하단 것으로 C에서 제공하는 직교성이라 볼 수 있습니다.

##### 직교성에 의한 오류

그러나 이러한 코드는 x와 y의 값을 비교한 결과가 조건문을 판별하는 것이 아니라, x에 할당된 결과로 조건문을 판별하게 되어 프로그래머가 의도하지 않은 결과를 유발할 수 있습니다.

직교성을 높이기 위해선 코드의 목적과 의도를 명확하게 표현해야합니다. 위 코드의 경우 할당 연산자가 아닌 동등 연산자(==)를 사용하는 것으로 코드의 의미를 명확하게 전달하며 오류를 예방할 수 있습니다.

<br/><br/><br/>

---

```c
struct exampleS
{
    int x;
    int y;
};

exampleS modifyS(int Px, int Py) {
    exampleS s;
    s.x = Px;
    s.y = Py;
    return s;
}

int main(void) {
    exampleS s1;
    s1 = modifyS(1,2);

    exampleS s2;
    s2.x=3;
    s2.y=4;

    return 0;
}
```

위 코드는 구조체의 멤버 변수에 직접 접근하여 값을 수정하는 예시입니다.
C에서는 `modifyS`함수를 사용해 구조체의 멤버 변수 x,y를 수정하거나 `main`함수 내에서 바로 구조체의 멤버 변수를 수정할 수 있습니다. 이러한 동작은 C에서 제공하는 직교성에 해당합니다.

##### 직교성에 의한 오류

그러나 이러한 방식으로 구조체의 멤버 변수를 직접 수정하는 행위는 해당 구조체를 사용하는 다른 코드의 실행에 영향을 줄 수 있으며, 프로그래머가 의도하지 않은 결과를 유발할 수 있습니다.

이러한 오류를 방지하기 위해선 구조체 내부에 x,y값을 수정하는 멤버 함수를 작성해 해당 멤버 함수를 통해 구조체의 멤버 변수를 간접적으로 접근하여 수정하는 방식을 사용해야 합니다.
