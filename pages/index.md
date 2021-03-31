---
layout: home
title: Home
nav_order: 1
has_children: true
permalink: /
has_toc: false
---

<h1> <span class="bteam">7842B</span> Digital Engineering Journal </h1>

This is the **2020/2021 Digital Engineering Journal** for team **7842B**{: class="bteam"}, during the **VEX Change Up** season.

We are a two-person team, consisting of:

- **Sawyer McClellan** - Builder
- **Theo Lemay** - Programmer

This is a joint digital journal, with sections for building and programming.

[Build Journal]({{site.url}}/build){: .btn }
[Programming Journal]({{site.url}}/programming){: .btn }

Parts of the code used for the robot is built on previous years work, so relevant journal entries are kept in archive.

[Programming Archive]({{site.url}}/archive){: .btn }

---

## All Posts

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% for post in site.posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date_to_long_string }}
	</li>
	{% endfor %}
</ul>
