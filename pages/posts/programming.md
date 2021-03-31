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

My primary role on the team is to design driver controls, design autonomous motion control algorithms, and design and develop a 1-minute "Autonomous Skills" routine. I also assist with building and driving of the robot.

In the Change Up season, I have been investing the majority of my time on **three major topics**:
- Designing controls for the robot's intakes to make it as easy as possible to drive
	- Developing an intuitive and versatile **driver control scheme**
	- Implementing a **statemachine controller** that combines driver and sensor input to achieve smooth and intuitive ball handling
	- Using sensors to **automate features** like "autopooping", which is the automatic disposal of unwanted balls   
- Designing **motion algorithms** for the robot to navigate during autonomous
	- Developing smooth, repeatable, and powerful motion controllers, for **fast and optimal** movement
	- Using sensors to **improve reliability**
- Planning an autonomous routine, making best possible use of the robot

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
