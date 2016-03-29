---
layout: default
---

{% for post in site.categories.top %}

# [{{ post.title }}]({{ post.url }})

{{ post.content }}

{% endfor %}

<br/>
<br/>

{% for post in site.categories.tech limit:1 %}

# [{{ post.title }}]({{ post.url }})

{{ post.content }}

{% endfor %}
