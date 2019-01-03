---
layout: default
title: Links
---
{% for link in site.data.links %}
<div class="post">
<a href="{{ link.url }}">{{ link.name }}</a>
<p>{{ link.description }}</p>
</div>
{% endfor %}
