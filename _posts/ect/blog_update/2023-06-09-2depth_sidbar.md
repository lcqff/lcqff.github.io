---
layout: post
title: "[Gitblog] 2-depth 사이드바 제작"
author: 파녀미
categories: 블로그_업데이트
banner:
  image: https://github.com/lcqff/lcqff.github.io/assets/71930280/cab142d6-e1d7-4e36-bbec-951baa71837a
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: gitblog
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

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/839c6fac-3520-4e63-8580-83d2f9ad0654)

내가 사용하는 테마인 Yat은 위와 같은 사이드바를 제공해준다.

당연히 마음에 들지 않았다….😌 특히 카테고리 종류가 늘어날수록 지저분해 보여 항상 사이드바를 2-depth로 수정하자고 생각하고 있었다.

그러나 jekyll를 잘 몰랐기에 jekyll 테마의 코드는 해독이 어려웠고; 사이드바를 만드는 방법을 열심히 구글링해봤지만 2-depth의 사이드바를 적용한 얘시를 찾기 어려웠다.

특히 대부분 다른 테마를 사용하고 있었고, 그로인한 파일구조가 달랐기에 내 블로그에 적용하기는 더 어려워보였다🥲

[[Github 블로그] minimal-mistake 블로그 카테고리 만들기 (+ 전체 글 수)](https://ansohxxn.github.io/blog/category/)

그때 발견한 게시글!

이거라면 내 블로그에도 적용할 수 있을 것 같았다. 많은 부분을 위 블로그를 참고했다! 정말 훌륭한 게시글!

그러나 위에서 서술했듯 나는 Yat 테마를 사용하고 있었고, 위 게시글에서 사용한 테마와는 다른 점이 참 많았다. + jekyll과 Liquid를 잘 몰랐기 때문에 정말 많은 부분을 감으로 시도해야했다.

이 게시글은 2-depth 사이드바를 블로그에 적용하는 방법과 그로 이해하게 된 약간의 원리…?를 서술하는 게시글이 될 것 같다.

위 게시글을 참고한 내용이므로 위 게시글에서 서술된 많은 부분이 생략될 예정이다. 게시글을 먼저 읽고오는 것을 추천한다.

---

## 파일 생성

2-depth 사이드바를 만들기 위해서 우선 아래 파일을 추가해야한다.

- **\_includes/category_list_main.html**
    <div class="callout">
    <div>🔎</div>
    <div>
        _includes 폴더 내부의 파일들은 Liquid에서 include 명령을 통해 가져올수 있다.<br/>
        <code>&#123;% include sidebar/category_list_main.html %}</code>
    </div>
    </div>
  - 사이드바를 구성하는 html 파일이다. 참고한 블로그의 파일명은 **nav_list_main**였으나 수정했다.
  - html 파일 내용은 위 블로그에서 긁어와 거의 수정하지 않았다.
  - 위 블로그에선 html 확장자를 붙이지 말라고 했는데 나는 붙여도 정상적으로 작동했다. 기존 사이드바 파일도 html 확장자가 붙어있어 확장자를 붙이는 식으로 통일했다. <br/><br/>
- **\_posts/categories/category-{카테고리 이름}.md**

  - <p style="color: red; font-weight:bold;" >나는 이 파일을 사용하지 않은 거 같다</p>
  - 아마 해당 카테고리 내의 파일만 보여주는 기능도 담당하는 거 같은데… 사이드바 만드는 것만으로도 너무 진이 빠져서 다음에 사용해 보는 걸로…^^;
  - 작은 카테고리 별로 categories 폴더 안의 md 파일로 저장하는 기능을 한다.

<br/>
<br/>
이때 **_posts/categories/category-{카테고리 이름}.md**에서 include 하는 내용을 <code>include <b>views/post-item.html</b></code> 로 수정했다.

<pre><code>
&#45;&#45;&#45;
title: "블로그 Jekyll"
layout: framework
permalink: categories/jekyll
author_profile: true
sidebar_main: true

---

&#123;% assign posts = site.categories.Jekyll %}

&#123;% for post in posts %}
&#123;% include <b>views/post-item.html</b> %}
&#123;% endfor %}
</code></pre>

테마가 다르기 때문에 비슷한 기능을 하는 파일도 파일명이 다르기 때문인데, 이때 **`views/post-item.html`**은
![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/ec0f8301-b7a1-4739-8088-62a9ab0dbb6b)
와 같이 게시글을 같이 표현해주는 html 파일이다.

내 블로그 테마에선 기존에 제공해주던 기능이었기에 해당 기능을 제공해주는 파일을 찾아 대신 include 해주었다.
(자신의 블로그에서도 위와 비슷한 기능을 제공하나 파일을 찾지 못하겠다면 개발자 도구의 select 기능을 사용해 class 이름을 확인해보도록 하자…)

## include **nav_list_main**

기존 사이드바를 제공하는 HTML 파일 제일 아래에 아래와 같은 코드를 추가한다.

<pre><code>
&#123;% if <b>site.sidebar_main</b>%} 
    &#123;% include category_list_main.html %} 
&#123;% endif%}
</code></pre>

- <code><b>&#123;% if site.sidebar_main%}</b></code>
  > \_config.yml 파일의 sidebar_main 값이 true인지 판단한다.
  - 참고한 블로그에서는 \_config.yml 파일의 값을 가져오기 위해 page를 사용한 거 같은데(추정) page가 아니라 site를 사용해야 들고와졌다. [참고](https://stackoverflow.com/questions/12062766/jekyll-use-config-yml-values-in-the-html)
  - \_config.yml에 아래 코드를 추가했다.
    ![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/6b1c0754-dc0d-4251-aa0b-9c818e730544)
  - 만일 참고한 블로그처럼 default 아래에 sidebar_main을 추가하고 싶다면 위 코드를 <code>&#123;% if site.defaults.values.sidebar_main%}</code>으로 수정하고 \_config.yml을 다음과 같이 수정하면 된다.
    ![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/8f69bb85-d080-4e2f-a4b3-09c929b1d01b)
    <br/>
- <code><b>&#123;% include category_list_main.html %}</b></code>
  - include를 사용해 **category_list_main.html** 내용을 가져왔다.

적용이 안된다면 if 문을 빼고 include 문만 사용해보는 걸 권한다. 적어도 어느쪽이 문제인지는 알 수 있을 거다…^^;

## 적용된 모습

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/7964ae21-3ac3-448f-9fd7-e83a0f79b1ac)

헤더에 `categories: Jekyll`가 적용된 게시글들이 <code>&#123;% assign posts = site.categories.Jekyll %}</code> 코드에 의해 **categoty-jekyll.md** 파일이 표시한 Jekyll이라는 작은 카테고리 안으로 들어간걸 확인할 수 있었다.

<pre><code>
    &#60;span class="nav__sub-title">Jekyll</span>
            &#60;ul>
                &#123;% for category in site.categories %}
                    &#123;% if category[0] == "Jekyll" %}
                        &#60;li>	&#60;a href="/categories/jekyll" class="">Jekyll ({{category[1].size}})</a></li>
                    &#123;% endif %}
                &#123;% endfor %}
            &#60;/ul>
</code></pre>

참고로 내 category_list_main.html은 위와 같이 수정해뒀다.

## 사이드바 교체

이 정도면 다 끝났다! 내 블로그에 맞게 수정하기만 하면 된다.

참고로 내 블로그에서 사용하고 있는 테마는 sidebar를 여러개 사용한다. tag, home, archives에 들어가면 보이는 사이드바의 모습이 각각 다 다르다^^;

<div class="imageRow">
  <div class="captionedImg">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/a5d9f1ed-6aae-45c9-976f-568d30ffe3cc">
    <p >archives의 sidebar</p>
  </div>
  <div class="captionedImg">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/ec4c3b92-e4f4-4c5c-ada3-088777f5dad8">
    <p >home의 sidebar</p>
  </div>
  <div class="captionedImg">
    <img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/d7bdd615-bbcb-4f7f-adff-464db6b1f02d">
    <p >tag의 sidebar</p>
  </div>
</div>
그래서 사이드바마다 파일이 따로 존재한다^^;

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/f7310326-ca36-4def-a125-cdfbe61fe8c7)

main이 되는 사이드바, 즉 home에서 보이는 사이드바는 common-list.html이며, 새로운 사이드바 코드는 임시로 common-list.html의 제일 아래쪽에 추가해두었다.

이제 home에 접속했을 시 **common-list.html** 대신 **category_list_main.html**이 뜨도록 수정해보자.

common-list.html을 검색해보니 common-list.html을 include하는 파일은 **category-list.html**이란 걸 알 수 있었다.

<pre><code>
&#123;%- include functions.html func='log' level='debug' msg='Get categories value'
-%} 

&#123;%- include functions.html func='get_categories' -%} 
&#123;% assign categories= return %} 

&#123;% assign keys = categories %} 
&#123;% assign field = 'categories' %} 
&#123;% assign url = '/categories.html' | relative_url %} 
<b>&#123;% include sidebar/common-list.html %}</b>
</code></pre>

기존 category-list.html

<pre><code>
&#123;%- include functions.html func='log' level='debug' msg='Get categories value'
-%} 

&#123;%- include functions.html func='get_categories' -%} 
&#123;% assign categories= return %} 

&#123;% assign keys = categories %} 
&#123;% assign field = 'categories' %} 
&#123;% assign url = '/categories.html' | relative_url %} 
<b>&#123;% if site.sidebar_main%} 
    &#123;% include sidebar/category_list_main.html %}
&#123;%endif %}</b>
</code></pre>

category-list.html를 위와 같이 수정해주었다.

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/399b251f-b199-4509-8c36-feb02455cd82)

기존에 뜨던 사이드바 대신에 수정한 사이드바가 뜨는 것을 확인할 수 있었다!

당연하지만 tag와 archives에 사용되는 사이드바는 교체되지 않았다.

## 변수 추가

이전에 있던 사이드바를 삭제하기엔 아깝다.

현재 코드는 \_config.yml의 sidebar_main의 값이 true일 때 수정된 사이드바가 뜨도록 구성되어있다.

그러면 기존의 사이드바도 변수를 따로 만들어 해당 값이 true일때만 출력되도록 수정해보자.

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/623c8342-8637-44cb-a3a9-026425486e7d)

\_config.yml에 **sidebar_defualt** 값을 추가했다.

<pre><code>
&#123;%- include functions.html func='log' level='debug' msg='Get categories value' -%}

&#123;%- include functions.html func='get_categories' -%}
&#123;% assign categories = return %}

&#123;% assign keys = categories %}
&#123;% assign field = 'categories' %}
&#123;% assign url = '/categories.html' | relative_url %}
&#123;% if site.sidebar_main%} 
    &#123;% include sidebar/category_list_main.html %}
&#123;%endif %}
<b>&#123;% if site.sidebar_defualt%} 
    &#123;% include sidebar/common-list.html %}
&#123;%endif %}</b>
</code></pre>

**category-list.html**에 위와 같은 코드를 추가했다. sidebar_defualt의 값이 true일 경우에 common-list.html을 출력할 것이다.

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/e70bb31e-e9ec-4abf-b5a5-99a14035a48b)

이렇게 말이다.

제대로 작동함을 확인했으면 sidebar_defualt를 false로 적용해 필요할 때만 보이도록 만들자.

---

## css 수정

![image](https://github.com/lcqff/lcqff.github.io/assets/71930280/f6220ffa-40e2-4c0c-84e8-a56ad569bdba)

css를 손보고 카테고리도 추가해주었다.

이건 본인의 블로그에 맞게 수정하면 될 거 같다

이걸로 완성~~~!

접었다 폈다 할 수 있는 기능도 넣고 싶은데 이건 나중에….😅



## 중복 코드 삭제
6개월만에 리팩토링을 했다. 카테고리와 서브카테고리를 추가할 때 직접 사이드바 코드를 수정해야 하는 부분이 마음에 걸리기도 했고, 카테고리가 어떻게 구성되어 있는지 코드상으로 확인하기 어려웠기 때문이다. 

(해당 게시글)[https://lcqff.github.io/%EB%B8%94%EB%A1%9C%EA%B7%B8_%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8/2023/12/08/2depth_sidbar2.html] 참고