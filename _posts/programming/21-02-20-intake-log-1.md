---
title: Intake Log 1
parent: Intake Automation
---

I need to design a controller for the intakes.

Features:

- use sensors to automatically shut off rollers when they hold a ball during
  intaking
- automatically poop unwanted balls
- space out balls when double shooting
- execute actions like shoot, outtake, purge, etc

To do this, I always start by defining all possible subsystem states:

```cpp
enum class rollerStates {
  off, // all off
  offWithoutPoop, // all off

  out, // all backwards

  on, // all on
  onWithoutPoop, // all on without poop

  shoot, // all on but don't move intakes
  shootWithoutPoop, // all on but don't move intakes and don't poop

  intake, // load balls into robot. Disable rollers one by one, and auto poop
  intakeWithoutPoop, // load balls into robot. Disable rollers one by one

  poopIn, // poop while intaking
  poopOut, // poop while outtaking
  purge, // shoot while outtaking
  topOut, // only spin top roller

  deploy,
  timedPoop,
  timedShootPoop,
  spacedShoot, // all on but space the ball and no intakes
  topPoop, // bring down then poop
};
```
