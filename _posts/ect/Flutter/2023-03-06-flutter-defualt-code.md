---
layout: post
title: "flutter의 바탕이 되는 코드와 기본적인 위젯"
author: 사탄
categories: flutter
banner:
  image: https://user-images.githubusercontent.com/71930280/223092717-70063803-9f73-4500-9547-40e7c8085dc2.png
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: flutter 위젯
sidebar: []
---

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp()); //어플리케이션 메인 페이지를 이 안에 넣어주자.
}

class MyApp extends StatelessWidget { //이것이 앱 메인페이지가 된다.
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      //실제 코드가 들어가는 부분
    );
  }
}
```

위는 flutter의 제일 밑바탕이 되는 코드이다.<br/>
flutter는 기본적으로 '위젯'을 사용해 화면을 구성한다. 위젯은 대문자 이름 뒤에 소괄호가 붙는 형식을 가지고 있다.

텍스트,아이콘,이미지,네모박스 위젯의 가장 기본적인 사용법을 알아보겠다.

### 텍스트

![스크린샷 2023-03-06 185619](https://user-images.githubusercontent.com/71930280/223093586-acd91473-2a94-4bf7-b265-3e710588c0e6.png)

**Text('안녕')**

### 아이콘

![스크린샷 2023-03-06 200755](https://user-images.githubusercontent.com/71930280/223093907-f0ab3042-718b-4570-8d6d-247b20e0bc2e.png)
**Icon(Icons.shop)**

### 이미지

![스크린샷 2023-03-06 190208](https://user-images.githubusercontent.com/71930280/223093916-e8421324-ec10-4d73-979b-fb80d581be34.png)
**Image.asset('경로')**<br/>
메인 폴더 아래에 asset폴더를 만들어 이미지를 집어넣자. 이미지를 사용하려면 **pubspec.yaml** 파일 내에 이미지를 등록해야한다.

```yaml
# The following section is specific to Flutter packages.
flutter:
  assets:
    - assets/
```

flutter 항목 내에 assets 폴더를 등록해줬다.  
![스크린샷 2023-03-06 200959](https://user-images.githubusercontent.com/71930280/223094350-749edd80-52db-433a-8898-490715ac04aa.png)
**Image.asset('assets/olli-the-polite-cat.png')**

### 네모박스

Container() 혹은 SizedBox()를 사용해도 상관없다. <br/><br/>
**Container(width: 50, height: 50, color: Colors.lightBlue,)**<br/>
flutter에서 숫자의 단위는 LP를 사용한다. 50LP==1.2cm정도<br/>
그러나 home에 해당 코드를 작성하면 실행해 보았을때 Box가 전체 화면을 채우고 있음을 확인할 수 있다.<br/>

박스의 크기를 지정해주기 위해서는 어디서부터 시작할지를 정해줘야한다. 이는 부모가 결정한다.

### 기준점 설정

**Center()** <br/>
자식 위젯의 기준점을 중앙으로 설정해준다.
![스크린샷 2023-03-06 192609](https://user-images.githubusercontent.com/71930280/223094643-b6d0f9be-5817-470b-bb8e-7086ad1a4372.png)

```dart
     home: Center(
        child:  Container(width: 50, height: 50, color: Colors.lightBlue,),
      )
```

이렇게 child라는 이름의 파라미터를 설정해주는 것으로 위젯 내부에 위젯을 넣을 수 있다.
