---
title: Skills
permalink: programming/skills
parent: Programming
has_children: true
has_toc: false
---

# Programming Skills

I need to find the best programming skills routine. Considerations:

- point turns are slow and prone to error
- reducing the number of separate motions and optimizing for long, smooth
  motions is ideal
- the locations where the robot can dispose blue balls (poop) must be carefully
  selected to not collide with other balls
- goals can be used to realign the robot

## Template Maps

![]({{site.url}}/assets/images/skills-bare.png)
![]({{site.url}}/assets/images/skills-labeled.png)

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Skills" | reverse %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %u, %Y" }}
	</li>
	{% endfor %}
</ul>
