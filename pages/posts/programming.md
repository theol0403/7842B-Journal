---
layout: default
title: Programming
nav_order: 3
permalink: programming
search_exclude: true
has_children: true
has_toc: false
---

# Programming Journal

Hello! I'm **Theo Lemay**, and this is my digital programming journal for the 2020/2021 **Change Up** season.

My primary role on the team is to design **driver controls**, design autonomous **motion control algorithms**, and design and develop a 1-minute **autonomous skills routine**. I also assist with building and driving.

In the Change Up season, I have invested the majority of my time on three major tasks:

### Intake Control

- Develop an intuitive and versatile **driver control scheme** for the **robot intakes** that make it easy to drive.
- Implement a **statemachine** controller that combines driver and sensor input to achieve smooth and intuitive ball handling.
- Use sensors to **automate features** like **autopooping**, which is the automatic disposal of unwanted balls.

[Intake Control]({{site.url}}/programming/intake){: .btn }

### Motion Control

- Develop smooth, repeatable, and powerful motion algorithms, for **fast and optimal** navigation during autonomous.
- Use sensors to **improve reliability**, like **localization**, **vision seeking**, and **velocity control**.

[Motion Control]({{site.url}}/programming/motion){: .btn }

### Programming Skills

- Design and implement **programming skills runs** that are consistent and fast.

[Programming Skills]({{site.url}}/programming/skills){: .btn }

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Programming" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date_to_long_string }}
	</li>
	{% endfor %}
</ul>
