---
title: V3 Skills
parent: Skills
nav_order: 3
---

## New routine

Here are a few ways we can shave off time and make it possible to get the last 4 balls:

- Robot V3 can intake the blue balls from the center tower rather than needing to poke them out. This can save us about 8 seconds.
- Spending less time at each goal and perfecting the shoot/intake timing can save us about 5 seconds.
- Adding a hood catapult to the robot can allow us to quickly score our preload, saving us about 3 seconds
- Redesigning the entire autonomous to be more efficient, with the last 4 balls in mind, and reducing point turns, can save us about 10 seconds.

For the last point, the more complex the motions are (driving while turning, complex arcs), the more optimal the routine can be.

One possible routine:

![](images/skills-planning-new.png)

Where **B** is the start, **E** is the end, <span style="color: #f01b44; font-weight: bold">red</span> is translation, <span style="color: #138ffb; font-weight: bold">blue</span> is front-facing direction, <span style="color: #26d761; font-weight: bold">green</span> is point turns, and <span style="color: #64e6e7; font-weight: bold">light blue</span> is potential "poop" locations. The <span style="color: #ffda3a; font-weight: bold">star</span> is a hood catapult. In this run, there are 11-12 point turns.
