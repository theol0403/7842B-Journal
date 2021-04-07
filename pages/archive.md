---
title: Archive
permalink: archive
nav_order: 100
has_children: true
has_toc: false
---

# Tower Takover Archive

This is an archive of old entries from last year. A lot of code and concepts is
reused in the current year.

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Archive" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %d, %Y" }}
	</li>
	{% endfor %}
</ul>
