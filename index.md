---
layout: default
title: Home
---
{% for post in site.posts %}
<div class="post">
	<h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
	<div class="meta">{{ post.date | date: '%d %B %Y' }}</div>
	{{ post.excerpt }}
</div>
{% endfor %}
