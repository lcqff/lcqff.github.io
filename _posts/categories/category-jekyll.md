{% assign posts = site.categories.Jekyll %}

{% for post in posts %}
{% include views/post-item.html %} {
% endfor %}
