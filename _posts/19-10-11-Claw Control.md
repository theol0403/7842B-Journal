---
title: "Claw Control"
parent: Programming
---

Today I had to figure out claw mapping and control. Our claw has an individual motor for each side of the claw, so I decided to keep the two sides completely separate. I wrote a statemachine that is used for both sides of the claw. 
When the driver releases buttons that either open or close the claw, it brakes the claw to prevent it from coasting.

```cpp
enum class clawStates { off, close, open, clamp, release, brake };
class Claw : public StateMachine<clawStates> {
  // ...
};
```

Here are the states:

```cpp
case clawStates::off: claw->moveVoltage(0); break;
case clawStates::close: claw->moveVoltage(12000); break;
case clawStates::open: claw->moveVoltage(-12000); break;
case clawStates::clamp:
  pid->setTarget(50);
  claw->moveVoltage(pid->step(claw->getPosition()) * 12000);
  break;
case clawStates::release:
  pid->setTarget(-200);
  claw->moveVoltage(pid->step(claw->getPosition()) * 12000);
  break;
case clawStates::brake: claw->moveVelocity(0); break;
```

Finally, here is the driver control layout for a single claw:

```cpp 
if (mDigital(L2) && mDigital(L1)) {
  leftClawState = clawStates::off;
} else if (mDigital(L2)) {
  leftClawState = clawStates::clamp;
} else if (mDigital(L1)) {
  leftClawState = clawStates::release;
} else {
  leftClawState = clawStates::brake;
}

if (leftClawState != lastLeftClawState) {
  Robot::clawLeft()->setState(leftClawState);
  lastLeftClawState = leftClawState;
}
```

