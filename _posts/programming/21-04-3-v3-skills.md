---
title: V3 Skills
parent: Skills
nav_order: 3
---

<!-- prettier-ignore-start -->
## Table of contents
{: .no_toc .text-delta }
1. TOC 
{:toc}

<!-- prettier-ignore-end -->

Previously, our routine was only designed to score a maximum of 122 points (out
of 126), purposefully excluding 4 of the balls. It looked something like this:

We are only slightly under time, so in order to get the last 4 balls, something
needs to be different. Simply modifying the current routine to grab the last 4
balls is not possible.

## New routine

Here are a few ways we can shave off time and make it possible to get the last 4
balls:

- Robot V3 can intake the blue balls from the center tower rather than needing
  to poke them out. This can save us about 8 seconds.
- Spending less time at each goal and perfecting the shoot/intake timing can
  save us about 5 seconds.
- Adding a hood catapult to the robot can allow us to quickly score our preload,
  saving us about 3 seconds
- Redesigning the entire autonomous to be more efficient, with the last 4 balls
  in mind, and reducing point turns, can save us about 10 seconds.

For the last point, the more complex the motions are (driving while turning,
complex arcs), the more optimal the routine can be.

One possible routine:

![](images/skills-planning-new.png)

Where **B** is the start, **E** is the end,
<span style="color: #f01b44; font-weight: bold">red</span> is translation,
<span style="color: #138ffb; font-weight: bold">blue</span> is front-facing
direction, <span style="color: #26d761; font-weight: bold">green</span> is point
turns, and <span style="color: #64e6e7; font-weight: bold">light blue</span> is
potential "poop" locations. The
<span style="color: #ffda3a; font-weight: bold">star</span> is a hood catapult.

In this run, there are 11-12 point turns.

## Ambitious Plan

Here is a new ambitious plan which only has **4** point turns!

![](images/skills-sensors.png)

Where <span style="color: #e11720; font-weight: bold">red</span> is forward,
<span style="color: #135bb5; font-weight: bold">blue</span> is backward,
<span style="color: #34d07a; font-weight: bold">green</span> is strafing,
<span style="color: #9141ac; font-weight: bold">purple</span> is heading,
<span style="color: #ea9e50; font-weight: bold">orange</span> is poop locations,
and <span style="color: #cdab8f; font-weight: bold">beige</span> is locations
for line-sensor alignment.

It requires:

- holonomic motion profiling (turning while driving)
- consistent velocity following
- line sensor adjustment

As long as I achieve these things in code the routine should be doable.

## Update Apr 2

I drove the autonomous routine and verified that it is in fact possible to
execute. Onwards with the plan!
