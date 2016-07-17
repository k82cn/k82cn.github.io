---
layout: default
---

<ul>
  {% for post in site.posts %}
    <li style="margin:5px 5px 5px 5px; list-style-type:none;">
      <span style="color: #aaa; font-family: Monaco, monospace; font-size: 80%;display:-moz-inline-box;display:inline-block;">{{ post.date | date_to_string }}</span>
      &raquo;
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

