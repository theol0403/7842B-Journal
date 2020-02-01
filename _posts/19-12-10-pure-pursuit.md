---
title: Pure Pursuit
parent: lib7842
nav_order: 3
---

Pure pursuit is a path following algorithm. Using odometry information and a path, pure pursuit controls how the robot should move to follow the path.

Pure pursuit has a few advantages:
- Motion profile capabilities: acceleration and velocity constraints
- Expressive arcs and curves
- Dynamic and feedback-based correction

A great paper on adaptive pure pursuit can be found [here](https://www.chiefdelphi.com/t/paper-implementation-of-the-adaptive-pure-pursuit-controller/166552/).

## How it works

Pure pursuit works by finding a *lookahead* on the path. It does this by finding the intersection of a circle of a given radius with the path. Then, the robot seeks the lookahead by calculating curvature and velocity to reach the lookahead.

![]({{site.url}}/assets/images/pure-pursuit.png)

## Implementing Pure Pursuit

To test pursuit, I wrote a full simulation using javascript. I was able to implement all the logic and optimization in that controlled environment. Implementing algorithms in javascript is very useful, as it provides a visual way of testing the algorithm and the code can translate to C++ quite easily. This algorithm has been in progress since June 2019.

<object width="100%" height="500" data="{{site.url}}/assets/demos/pathGeneration/index.html"> 
</object> 

Here are the controls:
- Click to place node 
- Click and drag to move node
- Right-click node to delete 
- Right-click and drag to delete selection
- Scroll to change angle

Here are the sliders:
1. Sample resolution: how many points should be generated from the path formula
2. Smoothing constant: how smooth should the connections between segments be
3. Lookahead distance: how far along the path should the robot look

## Code

Here is the lib7842 implementation of the pure-pursuit algorithm:

```cpp

```