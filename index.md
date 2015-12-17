---
layout: default
---

{% for post in site.categories.top %}
#[{{ post.title }}]({{ post.url }})

{{ post.content }}

<br/>
<br/>
{% endfor %}


{% for post in site.categories.tech limit:3 %}
#[{{ post.title }}]({{ post.url }})

{{ post.content }}

<br/>
<br/>
{% endfor %}

