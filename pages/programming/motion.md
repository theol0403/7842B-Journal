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

My focus in VEX has been developing such algorithms:

- 2018: [Tank drive odometry
  PID]({{ site.url }}{% link _posts/archive/19-11-15-odom-controller.md %})
  (taken from
  [here]({{ site.url }}{% link _posts/archive/19-08-19-robot-showcase.md %}#odometryautonomous))
- 2019: [X drive odometry
  PID]({{ site.url }}{% link _posts/archive/19-11-20-odom-x-controller.md %})
- 2019: [Pure
  Pursuit]({{ site.url }}{% link _posts/archive/19-11-25-pure-pursuit.md %})

All these algorithms can achieve various motions and require odometry. However,
they all have various weaknesses.

The main use of odometry is to unlock many possibilities of motion control. This
is because you now have access to much more information that you can feed into
the algorithms. For example, if you know the coordinates of the robot, and you
want to drive to some other coordinate, it is trivial to calculate position and
orientation error and then use PID-based algorithms to minimize that error.

Odometry, however, is a tricky task:

- it drifts a ton
- it requires a robot designed with perfectly calibrated tracking wheels
- it requires much tuning
- it is sensitive to rocking, vibrations, etc

Assuming the robot has perfectly tuned odometry, you still run into issues. This
is because anything directly using odometry is inherently a **closed loop
controller**. This means that the controller's **output** is a direct function
of the robot's **error** (calculated from **state**).

![]({{ site.url }}/assets/images/control.jpeg)

While this theoretically seems like a good thing, as the robot can use sensors
to correct for disturbances and errors, consider some realities:

- in VEX (especially skills), there is nothing that will interfere with the
  robot
- if something is wrong, misalligned, or stuck, odometry can't fix that
- using vex sensors for localization is not accurate, fast, or smooth

Instead, what ends up happening is that **odom introduces noise and variance**
into the controller, in a vicious cycle of tracking error -> correction -> more
error -> **unpredictable error**.

Is the definition of smooth, consistent control "do the exact same thing every
time", or "aggressively minimize error until the sensors say you are at your
destination"? I believe it is the former.

Is there a way to achieve the benefits of odometry (more complex and versatile
algorithms) without being overreliant on sensors? This is my objective for the
season.

---

<h2 class="text-delta">Table of contents</h2>

<ul id="markdown-toc">
	{% assign posts = site.posts | where: "parent", "Motion" %}
	{% for post in posts %}
	<li>
		<a href="{{ post.url | absolute_url }}">{{ post.title }}</a> 
		- {{ post.date | date: "%B %d, %Y" }}
	</li>
	{% endfor %}
</ul>
