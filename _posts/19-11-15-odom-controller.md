---
title: Odom Controller
parent: lib7842
nav_order: 1
---

In order to move to specific coordinates, I needed to design a versatile controller to allow me to tell the robot to drive to a point, using odometry information. This is quite a complex task, as it involves moving the robot in two dimensions while being efficient.

## Basic Movement

Once you know the position of the robot and where you want to move, the challenge is to actually move there. Here is a very simple algorithm to drive to a point:

```
1. Calculate angle to target
2. Turn to face the target
3. Calculate distance to target
4. Move distance to reach target
```

While this algorithm works decently well, it is quite slow and is not able to dynamically adjust while on-course. I instead wanted to make an algorithm that would curve toward the target, and calculate course adjustments on the fly.

### PID to heading and distance

The first algorithm I tried was PID. The distance and angle to the target would be sent to 2 PID controllers, and then the outputs would be combined. There were two problems with this method.

The first was that the algorithm had to be terminated when the robot reached the general vicinity of the target, or else the robot would start having a spasm. This is because the PID needs to have a negative input signal to be able to back up and settle. However, when calculating distance to a point (using Pythagoras), you can’t know when you overshoot the target. Therefore, the robot could only move forward, so when it reaches **and overshoots** the target, the angle to the target flips 180 degrees and the distance PID goes full throttle.

The second problem with this method is that it is not the most efficient. If the robot was perpendicular to the target, the distance PID would output full power, even though moving forward is the wrong thing to do.

![]({{site.url}}/assets/images/4c657dbc37e8ba9aa90ca55b61b76f05335459d1.png)

Instead, I wanted an algorithm that prioritized turning over moving, and that only moves when doing so would make the robot get nearer to the target.

## Adaptive PID Seeking
I posed this question: “If the robot is locked to its current heading, so it can only move forward/backward, how can it move in a straight line to get closest to the point as possible?”.

If the robot was perpendicular to the target, the answer would be 0. But as the robot rotates to face the target, the answer becomes more and more. Here are some images illustrating the question (the answer is the length of the red line):

![]({{site.url}}/assets/images/d540e20cfe7ba1395aed7a1ecdab796141a108c0.png)

After some research and help, I was able to implement the math for this. When doing distance PID on the output of these calculations, the robot was able to move much more efficiently. I also had to implement some logic to be able to drive backward. The reason this algorithm works great for settling is that if I turn off the angle PID when the robot is a certain distance away from the target, the adaptive distance PID brings the robot to a settled stop, giving a negative signal to back up.

To test this algorithm, I made a javascript simulation. You can see how the robot prefers turning over driving, and how it settles smoothly.

<object width="100%" height="300" data="{{site.url}}/assets/demos/adaptiveSeek.html"> 
    ![]({{site.url}}/assets/images/479e47a7761f01d48f5c41f49bfbaeaf4f75f1c8.gif)
</object>

## Settling
Every single autonomous motion has a settling period. However, I wanted the settling to be customizable for every single command. I wanted to be able to settle a few different ways:

* All PID controllers come to a rest
* All PID controllers get to some margin of error
* The robot gets to a certain requirement, such as a certain distance from a point or a certain angle

To do this, I created a parameter in each movement function that accepts a special kind of function called a `Settler`. Every loop, the movement function will ask the settler “am I settled yet?” and then the function will return true when it’s conditions are met.

Then, I created a few default settling functions and added the functionality to generate new settling functions on the fly. For example, here is a command with a full settle that comes to a stop:
```cpp
driveToPoint({1_ft, 1_ft}, 1, driveSettle);
```
If I wanted to make the robot exit the movement when it was 4 inches away from the target, I could say 

```cpp
chassis.driveToPoint({1_ft, 1_ft}, 1, makeSettle(4_in));
```

Here is the lib7842 implementation:
```cpp
/**
 * Function that returns true to end chassis movement. Used to implement different settling methods.
 */
using Settler = std::function<bool(const OdomController& odom)>;
```

## Turning
All turning is essentially the same motion. The only difference with all possible turns is the **goal** calculation and **movement method** (point, pivot, or arc). I wanted to write only one turning algorithm, and have all the implementations plug-in. 

Thus I used the same modular function pattern as the settling system and added parameters in the turning function to fulfill the angle calculation and movement method. Here are a few examples of a turn command:

```cpp
chassis.turn(angleCalc({1_ft, 1_ft}), pointTurn, turnSettle);
chassis.turn(angleCalc(90_deg), leftPivot, makeSettle(5_deg));
```

For simplicity, I also made a few helper functions - `turnToAngle(angle)`, `turnAngle(angle)`, and `turnToPoint(point)`, which all just call `turn(angleCalc)` internally.

Here is the lib7842 implementation:

```cpp
/**
 * Function that accepts a turning velocity and controls execution to the chassis. Used to implement
 * a point or pivot turn.
 */
using Turner = std::function<void(ChassisModel& model, double vel)>;

/**
 * Function that returns an angle for the chassis to seek. Examples can be an AngleCalculator that
 * returns the angle to a point, or an angle to an absolute angle.
 */
using AngleCalculator = std::function<QAngle(const OdomController& odom)>;
```

```cpp
  /**
   * Turn the chassis using the given AngleCalculator
   *
   * @param angleCalculator The angle calculator
   * @param turner          The turner
   * @param settler         The settler
   */
  virtual void turn(const AngleCalculator& angleCalculator, const Turner& turner = pointTurn,
                    const Settler& settler = defaultTurnSettler);
```

## AsyncAction
All the movement functions are blocking, but I still wanted to be able to trigger actions such as moving an arm in the middle of a movement. To do this I developed a system called AsyncAction, which requires a trigger (ex. min distance to point) and executes an action (ex. moves arm). In autonomous, used it to turn on the ball intake right when the robot reached the ball.

![]({{site.url}}/assets/images/44df1fcc5ac2962d52ed66e05f9c7f9b921a7242.png)

## Emergency Abort
There is nothing I dislike more than having the robot slightly off in autonomous, causing it to drive into a wall, and then being stuck there for the rest of the autonomous trying to push the wall over. Thus I made a system that detects when the robot is trying to move but is not moving. When it detects that the robot is stuck, it exits the current movement and goes on to the next one. With odometry, hopefully the robot will be able to continue the autonomous, because it still know its position even though it got stuck and had to abort a movement.