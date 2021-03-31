---
layout: default
title: Build
nav_order: 2
permalink: build
search_exclude: true
has_children: true
has_toc: false
---

{% assign build_posts = site.posts | where: "parent", "Build" %}

<h1>Build Journal</h1>

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% for post in build_posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date_to_long_string }}
	</li>
	{% endfor %}
</ul>

