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

I can then plug in that state into my handy [state machine
helper]({{ site.url }}{% link _posts/archive/19-10-16-statemachine.md %}):

```cpp
class Roller : public StateMachine<rollerStates, rollerStates::off> {
public:
  Roller(const std::shared_ptr<AbstractMotor>& iintakes,
         const std::shared_ptr<AbstractMotor>& ibottomRoller,
         const std::shared_ptr<AbstractMotor>& itopRoller,
         const std::shared_ptr<OpticalSensor>& itoplight,
         const std::shared_ptr<OpticalSensor>& ibottomLight,
         const std::shared_ptr<GUI::Graph>& igraph);

public:
  enum class colors { none = 0, red, blue };

  colors getTopLight() const;
  colors getBottomLight() const;

  bool shouldPoop();
  bool shouldShootPoop();
  bool shouldSpacedShoot();

  int getIntake();

  void initialize() override;
  void loop() override;

  std::shared_ptr<AbstractMotor> intakes {nullptr};
  std::shared_ptr<AbstractMotor> bottomRoller {nullptr};
  std::shared_ptr<AbstractMotor> topRoller {nullptr};
  std::shared_ptr<OpticalSensor> topLight {nullptr};
  std::shared_ptr<OpticalSensor> bottomLight {nullptr};
  std::shared_ptr<GUI::Graph> graph {nullptr};

  Timer macroTime;
  rollerStates macroReturnState = rollerStates::off;
  int macroIntakeVel = 12000;
};
```

Then, it is easy for me to design a driver control program:

```cpp
if ((mDigital(L1) || mDigital(R2)) && mDigital(R1)) {
  system(roller, on);
} else if (mDigital(R1) && mDigital(A)) {
  system(roller, on);
} else if (mDigital(A)) {
  system(roller, poopIn);
} else if (mDigital(R1)) {
  system(roller, shoot);
} else if (mDigital(L2)) {
  system(roller, out);
} else if (mDigital(B)) {
  system(roller, deploy);
} else if (mDigital(LEFT)) {
  system(roller, poopOut);
} else if (mDigital(DOWN)) {
  system(roller, topOut);
} else if (mDigital(L1) || mDigital(R2)) {
  system(roller, intake);
} else {
  system(roller, off);
}
```

Finally, I can implement the statemachine states.

```cpp
void Roller::loop() {
  Rate rate;

  while (true) {

    switch (state) {

      case rollerStates::off:
        if (shouldPoop()) continue;
      case rollerStates::offWithoutPoop:
        topRoller->moveVoltage(0);
        bottomRoller->moveVoltage(0);
        intakes->moveVoltage(0);
        break;

      case rollerStates::out:
        // if (shouldPoop(-12000)) continue;
        topRoller->moveVoltage(-12000);
        bottomRoller->moveVoltage(-12000);
        intakes->moveVoltage(-12000);
        break;

      case rollerStates::on:
      case rollerStates::shoot:
        if (shouldPoop()) continue;
        if (shouldShootPoop()) continue;
        [[fallthrough]];
      case rollerStates::onWithoutPoop:
      case rollerStates::shootWithoutPoop:
        if (shouldSpacedShoot()) continue;
        topRoller->moveVoltage(12000);
        if (getBottomLight() == colors::blue) {
          if (getTopLight() != colors::red) {
            bottomRoller->moveVoltage(-2000);
          } else {
            bottomRoller->moveVoltage(2000);
          }
        } else {
          bottomRoller->moveVoltage(12000);
        }
        intakes->moveVoltage(getIntake());
        break;

      case rollerStates::intake:
        if (shouldPoop()) continue;
        [[fallthrough]];
      case rollerStates::intakeWithoutPoop:
        if (getTopLight() != colors::none && getBottomLight() != colors::none) {
          topRoller->moveVoltage(0);
          if (getBottomLight() == colors::blue) {
            bottomRoller->moveVoltage(-2000);
          } else {
            bottomRoller->moveVoltage(0);
          }
          intakes->moveVoltage(12000);
        } else if (getTopLight() != colors::none) {
          // balance between raising ball to prevent rubbing and bringing ball too high
          topRoller->moveVoltage(1800);
          // slow down
          bottomRoller->moveVoltage(4000);
          intakes->moveVoltage(12000);
        } else {
          // balance between bringing ball too fast and accidentally pooping
          topRoller->moveVoltage(4000);
          if (getBottomLight() == colors::blue) {
            bottomRoller->moveVoltage(-2000);
          } else {
            bottomRoller->moveVoltage(12000);
          }
          intakes->moveVoltage(12000);
        }
        break;

      case rollerStates::poopIn:
        topRoller->moveVoltage(-12000);
        bottomRoller->moveVoltage(12000);
        intakes->moveVoltage(12000);
        break;

      case rollerStates::poopOut:
        topRoller->moveVoltage(-12000);
        bottomRoller->moveVoltage(12000);
        intakes->moveVoltage(-12000);
        break;

      case rollerStates::purge:
        topRoller->moveVoltage(12000);
        bottomRoller->moveVoltage(12000);
        intakes->moveVoltage(-12000);
        break;

      case rollerStates::topOut:
        topRoller->moveVoltage(12000);
        bottomRoller->moveVoltage(4000);
        intakes->moveVoltage(0);
        break;

      case rollerStates::deploy:
        topRoller->moveVoltage(12000);
        bottomRoller->moveVoltage(-800);
        intakes->moveVoltage(-12000);
        break;

      case rollerStates::timedPoop:
        topRoller->moveVoltage(-12000);
        bottomRoller->moveVoltage(12000);
        intakes->moveVoltage(macroIntakeVel);
        if (macroTime.getDtFromMark() >= 200_ms) {
          macroTime.clearMark();
          state = macroReturnState;
          continue;
        }
        break;

      case rollerStates::timedShootPoop:
        topRoller->moveVoltage(12000);
        bottomRoller->moveVoltage(0);
        intakes->moveVoltage(macroIntakeVel);
        if (macroTime.getDtFromMark() >= 400_ms) {
          macroTime.placeMark();
          state = rollerStates::timedPoop;
          continue;
        }
        break;

      case rollerStates::spacedShoot:
        topRoller->moveVoltage(12000);
        bottomRoller->moveVoltage(1000);
        intakes->moveVoltage(macroIntakeVel);
        if (macroTime.getDtFromMark() >= 10_ms) {
          macroTime.clearMark();
          state = macroReturnState;
          continue;
        }
        break;

      case rollerStates::topPoop:
        topRoller->moveVoltage(-12000);
        bottomRoller->moveVoltage(-12000);
        intakes->moveVoltage(macroIntakeVel);
        if (macroTime.getDtFromMark() >= 150_ms) {
          macroTime.placeMark();
          state = rollerStates::timedPoop;
          continue;
        }
        break;
    }

    rate.delayUntil(5_ms);
  }
}
```

This code is versatile because it is

- asynchronous
- responsive
- stateful (and the state can be easily retrieved and changed)
- common between driving and autonomous

It can:

- intake and hold balls
- shoot balls with spacing
- automatically engage poop states which auto return to the original state
- execute all the other actions needed

However, this code is slightly messy and inflexible, but I have an idea to
improve it in the next build log.
