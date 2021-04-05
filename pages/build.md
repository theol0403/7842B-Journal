---
title: Build
permalink: build
nav_order: 3
has_children: true
has_toc: false
---

# Build Journal

I'm **Sawyer McClellan**, and this is my **digital build journal** for the
2020/2021 **Change Up** season.

My primary role on the team is to **design and build** the robot and am the
**main driver**.

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Build" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %u, %Y" }}
	</li>
	{% endfor %}
</ul>
