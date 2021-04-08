---
title: Motion Control
permalink: programming/motion
parent: Programming
has_children: true
has_toc: false
nav_order: 3
---

# Motion Control

{% if page.author %} <span class="author">{{ page.author}}</span> {% endif %}

To execute any autonomous routine, the robot needs **movement algorithms**.
These are functions that use sensors, controllers, and kinematics to move the
robot somewhere.

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Motion Control" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %d, %Y" }}
	</li>
	{% endfor %}
</ul>
