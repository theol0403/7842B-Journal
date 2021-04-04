---
layout: default
title: Archive
nav_order: 100
has_children: true
permalink: archive
has_toc: false
---

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Archive" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %u, %Y" }}
	</li>
	{% endfor %}
</ul>
