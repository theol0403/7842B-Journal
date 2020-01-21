---
layout: post
title: "Lift PID"
---

Today I worked on the lift PID. I had to find a way to stabilize the sides.

Currently, here is the driver control mapping:

```cpp
bool moveSlow = mDigital(R1);

if (mDigital(L2) && mDigital(L1)) {
  liftState = liftStates::bottom;
} else if (mDigital(L1)) {
  liftState = moveSlow ? liftStates::upSlow : liftStates::up;
} else if (mDigital(L2)) {
  liftState = moveSlow ? liftStates::downSlow : liftStates::down;
} else {
  liftState = liftStates::hold;
}
```

When the driver presses `L2` or `L1`, the lift moves at full speed, and when the driver lets go, the lift tries to maintain the position where the button was let go. However, since there is an individual PID running on each side of the lift, it might try to maintain the lift to not be straight. To fix this problem, I averaged the two sides of the lift and used that for my PID target.

Do do this, I used `std::valarray` , which makes it easier to do math with a set of numbers:

```cpp
std::valarray<double> Lift::getPosition() const {
  return std::valarray<double>({lift[0]->getPosition(), lift[1]->getPosition()}) - startPos;
}
// calculate hold position by averaging
holdPos = getPosition().sum() / 2.0;
```

### Code summary

Here are the statemachine states in question:

```cpp
case liftStates::off:
  lift[0]->moveVoltage(0);
  lift[1]->moveVoltage(0);
  break;

case liftStates::up:
  lift[0]->moveVoltage(12000);
  lift[1]->moveVoltage(12000);
  break;

case liftStates::down:
  lift[0]->moveVoltage(-12000);
  lift[1]->moveVoltage(-12000);
  break;

case liftStates::upSlow:
  lift[0]->moveVelocity(50);
  lift[1]->moveVelocity(50);
  break;

case liftStates::downSlow:
  lift[0]->moveVelocity(-50);
  lift[1]->moveVelocity(-50);
  break;

case liftStates::hold:
  holdPos = getPosition().sum() / 2.0;
  state = liftStates::holdAtPos;
  break;

case liftStates::holdAtPos:
  pid[0]->setTarget(holdPos[0]);
  pid[1]->setTarget(holdPos[1]);
  lift[0]->moveVoltage(pid[0]->step(getPosition()[0]) * 12000);
  lift[1]->moveVoltage(pid[1]->step(getPosition()[1]) * 12000);
  break;

case liftStates::bottom:
  pid[0]->setTarget(0);
  pid[1]->setTarget(0);
  lift[0]->moveVoltage(pid[0]->step(getPosition()[0]) * 12000);
  lift[1]->moveVoltage(pid[1]->step(getPosition()[1]) * 12000);
  break;
```

