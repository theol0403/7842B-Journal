---
layout: home
title: Home
nav_order: 1
has_children: true
permalink: /
has_toc: false
---

# 7842B Digital Engineering Journal

Welcome to the **Digital Engineering Journal** for VEX team **7842B** in the **Change Up** season.

![]({{site.url}}/assets/images/20210330_162158.jpg){:height="40%" width="40%"}

We are a two-person team, consisting of:

- **Sawyer McClellan** - Builder
- **Theo Lemay** - Programmer

This is a joint digital journal, with sections for building and programming.

[Build Journal]({{site.url}}/build){: .btn }
[Programming Journal]({{site.url}}/programming){: .btn }

Parts of the code used for the robot is built on previous years work, so relevant journal entries are kept in archive.

[Tower Takeover Archive]({{site.url}}/archive){: .btn }

---

## All Posts

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% for post in site.posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %u, %Y" }}
	</li>
	{% endfor %}
</ul>