---
layout: default
parent: Home
title: Motion Algorithms
nav_order: 1
---

# Motion Algorithms

A big focus for this year has been motion algorithms for autonomous. Motion algorithms allow for a faster, easier, and more competitive autonomous. This year, we have built an **X-Drive** robot, which opens up a wide range of possibilities. Having a holonomic chassis means that we can have *heading-agnostic* movement functions, meaning that we can drive anywhere while turning at a completely independent angle.

### Odometry

To be able to have a consistent autonomous while not moving in linear or straight manners, we need robot localization. To do this, we used the odometry algorithm provided by team [5225A](https://www.vexforum.com/t/team-5225-introduction-to-position-tracking-document/49640) from Ontario. With the use of three freely-spinning **tracking wheels**, we are able to use trigonometry to calculate the absolute position of the robot in the field.

<img src="/assets/images/image-20191115144427806.png" style="zoom:50%;" />

To calculate absolute heading, we can use the following formula:

```
angleRadians = (leftInch - rightInch) / chassisWidthInch
```

### Motion

Knowing where the robot is at all times allows us to do some pretty complex maneuvers. It allows us to do the following actions:

- Turn to face Absolute Angle
- Turn to face Point
- Turn to face Relative Angle
- Drive a Distance
- Drive to a Point

This means that we can program using coordinates from a map of the field:

<img src="/assets/images/field planning.png" style="zoom: 15%;" />

### X-Drive

Having an X-Drive allows us to do some even more complex algorithms. To unlock this capability, I developed a function that allows me to tell the chassis to strafe at a certain angle and magnitude, using this math:

![](/assets/images/image-20191115150107968.png)

With this math, I can say:

- Drive to Point while facing Angle
- Drive Distance while facing Angle

Read our autonomous programs for a demonstration of this functionality.

