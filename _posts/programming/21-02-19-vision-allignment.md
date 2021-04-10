---
parent: Motion Control
nav_order: 6
---

## Motivation

Even though V1 skills worked decently well, the greatest point of failure was
picking up the balls.

This is because even though the linear motions were very smooth and consistent,
the PID IMU point turns have a few degrees of unpredictable error. One solution
is to employ the use of more sensors.

The VEX vision sensor is perfect for this task: align with the balls when
driving to increase consistency. If the vision sensor is angled down so the
tiles act as a background, there will be very little interference, and the
sensor does not need to be able to see very far.

## Filtering

The vision sensor is quite noisy and inconsistent. Also, working with the
objects concisely is a challenging task. Therefore, I developed an API for
lib7842 to make it easy to work with vision objects.

The code used on the robot is:

```cpp
// read the sensor
auto container = vision->getAll().sort(Vision::Query::area);

// remove noise (super small objects) and unusable (full screen) objects
container.remove(Vision::Query::area, std::less<double>(), 3000)
  .remove(Vision::Query::area, std::greater<double>(), 34000);

// clear the rendering surface on the screen
drawer->clear();
// draw the objects onto the screen
drawer->makeLayer()
  .withColor(LV_COLOR_RED, LV_COLOR_BLACK, RED)
  .withColor(LV_COLOR_BLUE, LV_COLOR_BLACK, BLUE)
  .draw(container);

// extract the red balls, and get the position of the largest object relative to the center of the screen
auto reds = container;
offset = reds.remove(Vision::Query::sig, std::not_equal_to<double>(), RED)
           .get(0, Vision::Query::offsetCenterX);

// extract the blue balls, and get the position of the largest object relative to the center of the screen
auto blues = container;
blueOffset = blues.remove(Vision::Query::sig, std::not_equal_to<double>(), BLUE)
               .get(0, Vision::Query::offsetCenterX);
```

## Motion Control

Next, I need to use the vision data to align the robot. To do this, I use the x
coordinate relative to the center of the screen and feed it into a p loop.

That P loop then gets combined with the velocities from the trajectory generator
to gently correct the motion.

It works great on the robot!

## Update Feb 24

At the competition, it worked very well! The robot was able to much more
consistently grab the balls. Here is the full run:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/xOU6WEcxbS0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Update Feb 30

The main issue with the vision seeking is that it introduces variability and
causes the endpoint of a trajectory to be unknown, especially if the robot had
to aggressively compensate to grab a ball.

We had the idea of modifying the correction to strafe instead of turn, which
would result in less damage being done on the final pose. This, along with the
newly improved trajectory generator, worked together very nicely:

<iframe width="560" height="315" src="https://www.youtube.com/embed/9Ip8POgMptk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
