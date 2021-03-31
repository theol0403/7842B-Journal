---
title: Motion Algorithms
parent: Programming
---

A big focus for this year has been motion algorithms for autonomous. Motion algorithms allow for a faster, easier, and more competitive autonomous, and are really fun to make.

This year, we have built an **X-Drive** robot, which opens up a wide range of possibilities for control. Having a holonomic chassis means that we can have _heading-agnostic_ movement functions, meaning that we can drive any direction while facing any angle. This opens up possibilities for turning while driving, and also makes driving to points much easier as the heading of the robot is independent from movement.

### Odometry

To be able to execute complex movements that are not straight lines, we need to have a way to track the robot's position. This both improves accuracy and makes writing algorithms much easier as we can always know the absolute position and angle of the robot in the field.

To do this, we used an odometry algorithm provided by team [5225A](https://www.vexforum.com/t/team-5225-introduction-to-position-tracking-document/49640). With the use of three freely-spinning **tracking wheels**, we are able to use trigonometry to calculate the absolute position of the robot in the field.

<img src="{{site.url}}/assets/images/image-20191115144427806.png" width="80%" />

{% comment %}
TODO: Robot Image
{% endcomment %}

A very useful trick we have learned is to calculate the robot's absolute heading, we can use the following formula:

```
angleRadians = (leftInch - rightInch) / chassisWidthInch
```

### Motion

Motion algorithms will be a work in progress throughout the year. I will be implementing them in lib7842, so I aim to have good programming practice when making them. I will be starting from last year's code which will make things easier.

Knowing where the robot is at all times opens up many options. For example, here are some commands we can use to build autonomous programs:

```cpp
turnToAngle(angle);
turnToPoint(point);
driveDistance(distance);
driveToPoint(point);
followPath(path);
```

Since we have an X-Drive robot, we can often combine driving and turning into a single action, so the turning functions are barely used.

### Planning

Since we use odometry and motion algorithms to tell the robot to go to certain coordinates, we can much more easily plan autonomous programs using a map. We overlaid a coordinate system over a top-down view of the field, so we can simply plot and record points:

<img src="{{site.url}}/assets/images/field planning.png" width="70%" />
