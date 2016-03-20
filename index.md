---
layout: default
---

{% for post in site.categories.top %}

<h1>[{{ post.title }}]({{ post.url }})</h1>

{{ post.content }}

<br/>
<br/>
{% endfor %}


{% for post in site.categories.tech limit:3 %}

<h1>[{{ post.title }}]({{ post.url }})</h1>

{{ post.content }}

<br/>
<br/>
{% endfor %}

