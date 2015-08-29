---
layout: default
---

  <ul>
    {% for post in site.categories.blogs %}
      <li style="margin:5px 5px 5px 5px;">
        <span style="width:90px;display:-moz-inline-box;display:inline-block;">{{ post.date | date_to_string }}</span>
        <span style="width:10px;display:-moz-inline-box;display:inline-block;">-</span>
        <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>

