---
layout: default
---

<h2><a href="{{ site.url }}/news">News</a></h2>

<ul>
  {% for post in site.categories.news limit: 5 %}
    <li style="margin:5px 5px 5px 5px;">
      <span style="width:90px;display:-moz-inline-box;display:inline-block;">{{ post.date | date_to_string }}</span>
      <span style="width:10px;display:-moz-inline-box;display:inline-block;">-</span>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

<h2><a href="{{ site.url }}/tech">Tech</a></h2>

<ul>
  {% for post in site.categories.tech limit: 5 %}
    <li style="margin:5px 5px 5px 5px;">
      <span style="width:90px;display:-moz-inline-box;display:inline-block;">{{ post.date | date_to_string }}</span>
      <span style="width:10px;display:-moz-inline-box;display:inline-block;">-</span>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

<h2><a href="{{ site.url }}/blogs">Blogs</a></h2>

<ul>
  {% for post in site.categories.blogs limit: 5 %}
    <li style="margin:5px 5px 5px 5px;">
      <span style="width:90px;display:-moz-inline-box;display:inline-block;">{{ post.date | date_to_string }}</span>
      <span style="width:10px;display:-moz-inline-box;display:inline-block;">-</span>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

<h2><a href="{{ site.url }}/docs">Docs</a></h2>

<ul>
  {% for post in site.categories.docs limit: 5%}
    <li style="margin:5px 5px 5px 5px;">
      <span style="width:90px;display:-moz-inline-box;display:inline-block;">{{ post.date | date_to_string }}</span>
      <span style="width:10px;display:-moz-inline-box;display:inline-block;">-</span>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

