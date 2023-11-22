---
layout: post
title: Test markdown
subtitle: Each post also has a subtitle
categories: 샘플
banner:
  image: https://user-images.githubusercontent.com/71930280/236198202-1a6ec792-f31d-4e2b-858b-5535b4e25634.png
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: CSS HTML P1Ch8
---


You can write regular [markdown](https://markdowntutorial.com/) here and Jekyll will automatically convert it to a nice webpage. I strongly encourage you to [take 5 minutes to learn how to write in markdown](http://markdowntutorial.com/) - it'll teach you how to transform regular text into bold/italics/headings/tables/etc.

**Here is some bold text**

## Here is a secondary heading

Here's a useless table:

| Number | Next number | Previous number |
| :----- | :---------- | :-------------- |
| Five   | Six         | Four            |
| Ten    | Eleven      | Nine            |
| Seven  | Eight       | Six             |
| Two    | Three       | One             |


### 이미지

How about a yummy crepe?

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg)

<!-- It can also be centered!

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg){: .center-block :} -->

#### 이미지 row 정렬
<div>
  <img src="https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg">
  <img src="https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg">
  <img src="https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg">
</div>
{: .imageRow}   

#### 이미지 캡션
  <div class="captionedImg">
    <img src="https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg">
    <p >yummy crepe</p>
  </div>


Here's a code chunk:

```
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

And here is the same code with syntax highlighting:

```javascript
var foo = function (x) {
  return x + 5;
};
foo(3);
```

And here is the same code yet again but with line numbers:

{% highlight javascript linenos %}
var foo = function(x) {
return(x + 5);
}
foo(3)
{% endhighlight %}

### Boxes

You can add notification, warning and error boxes like this:

#### Notification

{: .box-note}
**Note:** This is a notification box.

#### yello

{: .box-yello}
**yello:** This is a yello box.

#### Error

{: .box-error}
**Error:** This is an error box.

#### 콜아웃

<div class="callout">
  <div>:memo:</div>
  <div>
    <strong>JS 는 기본적으로 동기로 실행 → 실행이 끝나야 다음 코드가 실행된다.</strong><br/>
    JS 는 기본적으로 동기로 실행 → 실행이 끝나야 다음 코드가 실행된다<br/>
    JS 는 기본적으로 동기로 실행 → 실행이 끝나야 다음 코드가 실행된다
  </div>
</div>

### 텍스트
#### 텍스트 색상
<p class="red"> 붉은색</p>
<p class="blue"> 푸른색</p>


