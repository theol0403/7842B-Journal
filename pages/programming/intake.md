---
title: Intake Automation
permalink: programming/intake
parent: Programming
has_children: true
has_toc: false
nav_order: 2
---

# Intake Automation

{% if page.author %} <span class="author">{{ page.author}}</span> {% endif %}

One of my main roles as programmer is to design and implement robot automation
and interaction with driver controls.

My go-to solution for subsystem control is to create a [statemachine
controller]({{ site.url }}{% link _posts/archive/19-10-16-statemachine.md %}).

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Intake Automation" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %d, %Y" }}
	</li>
	{% endfor %}
</ul>
