---
layout: post
title: "flutter의 레이아웃 위젯"
author: 사탄
categories: flutter
banner:
  image: https://user-images.githubusercontent.com/71930280/228833469-fec65426-429a-4a5b-86c1-0696c854056f.png
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: flutter 위젯
sidebar: []
---

`MaterialApp()`은 flutter에서 이용할수 있는 위젯 클래스의 생성자 함수이다.<br/>
`MaterialApp()`을 사용하면 구글이 제공하는 테마를 사용해 앱의 기본적인 구성요소를 설정할 수 있다.<br/>
또한 `MaterialApp()`은 앱을 구성하는 다른 위젯들의 루트 위젯으로 사용된다. 따라서 일반적으로 `MaterialApp()`을 사용해 앱의 시작점이 정의한다.

MaterialApp()을 사용한 위젯들은 보통 구글 스타일이 강한데, 그것이 싫다면 아이폰 기본 앱 스타일의 테마를 제공하는 `Cupertino~()`를 사용할 수도 있다.<br/>
그렇다고 커스터마이징 위젯을 사용할 수 없는 건 아니고, MaterialApp()에서 스타일을 수정하면 된다. (MaterialApp()에서 유용한 기능들을 대부분 제공하니 처음부터 만들진 말 것)

### 상중하 배치

![스크린샷 2023-03-27 222241](https://user-images.githubusercontent.com/71930280/228836139-ed9a24a9-4c93-43da-8138-4022187045ea.png)

```dart
    return MaterialApp(
        home:  Scaffold(
            appBar: AppBar(), //상단 레이아웃, 네비게이션 바를 생성해줌
            body: Container(), //내용 레이아웃
            bottomNavigationBar: BottomAppBar(child: Text('dsdfs'),), //하단 레이아웃
        )
    );
```

### 가로,세로 배치

![image](https://user-images.githubusercontent.com/71930280/228838011-04c9b3aa-5b79-4c01-a66a-cd713a4606a6.png)

```dart
    return MaterialApp(
        home:  Scaffold(
          body: Row(
              children: const [ Icon(Icons.star),Icon(Icons.star),Icon(Icons.star),]
          ),
        )
    );
```

![image](https://user-images.githubusercontent.com/71930280/228838914-dff9bac9-152e-4041-a070-e6ab54d2d6a7.png)

```dart
    return MaterialApp(
        home:  Scaffold(
          body: Column(
              children: const [ Icon(Icons.star),Icon(Icons.star),Icon(Icons.star),]
          ),
        )
    );
```

flutter에서 child 아래에 여러 위젯을 넣고 싶을 땐 `Row()`나 `Column()`같은 레이아웃 위젯을 사용해 그룹화 해야한다.<br/>
`Row()`나 `Column()` 위젯을 사용해 여러 위젯을 가로, 혹은 세로 배치할 수 있다.

### 중앙정렬

![image](https://user-images.githubusercontent.com/71930280/228839525-8aab2650-d47a-4376-8bba-43e4e3caf59f.png)

```dart
    return MaterialApp(
        home:  Scaffold(
          body: Row(
              **mainAxisAlignment: MainAxisAlignment.center**,
              children: const [ Icon(Icons.star),Icon(Icons.star),Icon(Icons.star),]
          ),
        )
    );
```

`mainAxisAlignment`: 현재 축을 따라 정렬해준다. 위 코드는 Row의 내부에 있으므로 가로축을 기준으로 배치한다. <br/>
여러가지 방식으로 배치가 가능하다. 예를 들어 `mainAxisAlignment: MainAxisAlignment.spaceEvenly`를 사용하면 요소끼리 간격을 두고 중앙정렬 할 수 있다.

### 반대축으로 중앙 정렬

현재 축으로 중앙정렬 받고 거기에 더해 반대축으로 중앙정렬하는 것도 알아보자.
<br/>
![image](https://user-images.githubusercontent.com/71930280/228840842-beaacc7a-342e-4da2-816d-ce29554488f2.png)

```dart
    return MaterialApp(
        home:  Scaffold(
            body: Container(
              height: 400,
              child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  **crossAxisAlignment: CrossAxisAlignment.center**,
                  children: const [ Icon(Icons.star),Icon(Icons.star),Icon(Icons.star),]
              ),
            )      )
    );
```

`crossAxisAlignment: CrossAxisAlignment.center`를 사용하면 현재 축인 Row의 반대축을 기준으로 중앙정렬해준다.<br/>
이때 반대축을 기준으로 중앙정렬하기 위해서는 범위가 필요하다. 따라서 Container에 감싸 height나 width 성분을 넣어주어야 한다.
