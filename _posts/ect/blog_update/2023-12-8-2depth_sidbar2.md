---
layout: post
title: "[Gitblog] 2-depth 사이드바 제작 - 중복 코드 삭제"
author: pam
categories: 블로그_업데이트
tags: gitblog
---

## 이전 코드의 문제

<pre><code>
&#123;% assign sum = site.posts | size %}

&#60;sdiv>
  &#60;!--전체 글 수-->
  &#60;sdiv class="common-list">
    ...
  &#60;/div>

  &#60;!--카테고리-->
  &#60;div class="parent_cat">CS</div>
  &#60;div class="common-list">
    &#60;ul>

      &#123;% for category in site.categories %} &#123;% if category[0] == "CS" %}
      &#60;li>
        &#60;a href="{{ url }}#h-{{ category[0] }}">
          &#123;&#123; category[0] }}
          &#60;span>&#123;&#123; site.posts | where: field, category[0] | size }}</span>
        &#60;/a>
      &#60;/li>
      &#123;% endif %} &#123;% endfor %} 
      &#60;!-- 서브 카테고리가 추가될 때마다 위 블럭을 반복...-->

    &#60;/ul>
  &#60;/div>
  &#60;!-- 카테고리가 추가될 때마다 위 블럭을 반복...-->
&#60;/div>
</code></pre>


2 depth sidebar을 만들고 방치해둔지 6개월... liquid를 모르는 상태로 2-depth 사이드바를 만든 것 자체로 진이 빠져 리팩토링은 미뤄둔 상태였다.  
사실 이때까지만 해도 블로그에 카테고리가 그렇게 많진 않아서 카테고리를 만질 일이 없었는데, 이번에 다시 블러그를 시작하며 카테고리를 몇 개 추가하고 순서를 바꾸려고 하니 좀 귀찮다고 느껴졌다😅  
카테고리를 수정할 때 카테고리 코드가 아닌 '카테고리 구성'을 저장해둔 파일만 수정하면 편할거 같다는 생각이 들었다.🤔   


### 첫번째 시도 - JS
처음에는 JS를 사용하려 시도했다. liquid에 익숙하지 않아 자연스럽게 중복코드를 JS를 사용하는 쪽으로 생각이 기울었던거 같다.  

JS를 사용해 사이드바를 구현하는 것까진 성공했으나..... 

<pre><code>
&#60;span> &#123;&#123; site.posts | where: field, category[0] | size }}</span>
</code></pre>
이녀석이 도무지 구현 불가능했다😱  
site.posts는 말 그대로 블로그에 있는 '포스트'를 들고오는 것으로 보이는데, 위의 Liquid문법을 JS에서 사용할 수 없었고, site.posts 데이터를 html에서 JS로 넘겨주는 것 또한 불가능했다.  
`<span>`이 굉장히 애매한 위치에 끼어있던 터라 위 코드는 html에서 처리하고 JS에서 중복 코드를 끼워넣는 방식으로 하는 것도 불가능했다.  
멘붕...😂


### 두번째 시도 - Liquid
결국 JS를 사용하는 것은 포기하고 순수하기 html내에서 중복코드를 처리하기로 마음먹었다.  
JS를 사용할땐 이중 배열에 카테고리와 서브카테고리 값들을 저장할 계획이었는데, Liquid를 사용해보니 Liquid에서 배열을 흉내낼순 있으나 이중 배열의 사용은 불가능한 거 같았다.  
따라서 카테고리 형식을 저장할 다른 방법이 필요했다. 


Liquid 템플릿 언어에서는 `site.data`를 사용해 _data디렉토리에 저장된 외부 데이터를 참고할 수 있다. 예를 들어 _data에 main_category.yaml 파일이 있다면 `site.data.main_category`를 사용해 해당 데이터를 조회할 수 있다.  

```yaml
  - name: "CS"
    subcategories:
      - "CS"
      - "Algorithm"
  - name: "BackEnd"
    subcategories:
      - "Spring"
      - "Java"
      - "JPA"
      - "오류기록장"

      ...
      
  - name: "기타"
    subcategories:
      - "블로그_업데이트"
      - "회고"
      - "샘플"
```
_data 디렉토리 내에 main_category.yaml파일을 생성했다. 구성은 사이드바 내용과 순서를 그대로 옮겨왔다.  

<br>
이제 위 데이터를 가져와서 사이드바 코드를 작성하면......
<pre>
<code>
&#123;% assign sum = site.posts | size %}

&#60;sdiv>
  &#60;!--전체 글 수-->
  &#60;sdiv class="common-list">
    ...
  &#60;/div>

  &#60;!--카테고리-->
  <strong>
  &#123;%- assign categories = site.data.main_category -%}
  &#123;% for category in categories %}
  </strong>
  &#60;div class="parent_cat">&#123;&#123; category.name }}</div>
  &#60;div class="common-list">
    &#60;ul>
    <strong> &#123;% for subCategory in category.subcategories %}</strong>
        &#60;li>
          &#60;a href="{{ url }}#h-{{ subCategory }}">
            &#123;&#123; subCategory }}
            &#60;span>&#123;&#123; site.posts | where: field, subCategory | size  }}</span>
          &#60;/a>
        &#60;/li>
        &#123;% endfor %}
      &#60;/ul>
    &#60;/div>
&#123;% endfor %}
&#60;/div>
</code>
</pre>

이럴수가.. 너무 깔끔해졌다!
JS로 했어도 이정도로 깔끔한 코드는 안나왔을 텐데, JS에서 실패하고 삽질을 꽤 오래 했었지만 차라리 잘된 일인거 같다.😂