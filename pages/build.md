---
title: Build
permalink: build
nav_order: 3
has_children: true
has_toc: false
---

# Build Journal

I'm **Sawyer McClellan**, and this is my **digital build journal** for the
2020/2021 **Change Up** season!

My primary role on the team is to **design and build** the robot. I also am the
**main driver**.

This year, we have gone through **3** major iterations of the robot. The
relevant features and improvements of each iteration is categorized below:

[Robot V1]({{site.url}}/build/v1){: .btn } [Robot V2]({{site.url}}/build/v2){:
.btn } [Robot V3]({{site.url}}/build/v3){: .btn }

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Build" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %d, %Y" }}
	</li>
	{% endfor %}
</ul>
