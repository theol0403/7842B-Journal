---
layout: home
title: Home
permalink: /
nav_order: 1
has_children: true
has_toc: false
---

# 7842B Digital Engineering Journal

Welcome to the **Digital Engineering Journal** for VEX team **7842B** in the
**Change Up** season!

We are a two-person team, consisting of:

- **Theo Lemay** - Programmer
- **Sawyer McClellan** - Builder

This is a joint digital journal, with sections for building and programming.

[Programming Journal]({{site.url}}/programming){: .btn }
[Build Journal]({{site.url}}/build){: .btn }

![]({{site.url}}/assets/images/20210330_162158.jpg){:height="40%" width="40%"}

## Archive

Parts of the code used for the robot is built on previous years work, so
relevant journal entries are kept in archive.

[Tower Takeover Archive]({{site.url}}/archive){: .btn }

---

## Post Index

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% for post in site.posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %u, %Y" }}
	</li>
	{% endfor %}
</ul>
