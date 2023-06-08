---
title: "블로그 Jekyll"
layout: framework
permalink: categories/jekyll
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Jekyll %}
{% for post in posts %} {% include views/post-item.html %} {% endfor %}
