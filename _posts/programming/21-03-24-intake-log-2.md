---
parent: Intake Automation
---

<!-- prettier-ignore-start -->
## Table of contents
{: .no_toc .text-delta }
1. TOC 
{:toc}

<!-- prettier-ignore-end -->

## Preface

Upon analysis of the most optimal driver skills run, we realized the biggest
factor that was impeding our time: we were pooping too early. We had always
driven so that the robot would try to get rid of blue balls right away, which
would keep the intakes empty. However, the blue balls that were pooped was
causing pandemonium in the field, and knocking balls out of the way.

We realized that the best driving runs were repeatable and clean - dealing with
a randomly jumbled field was not a good strategy.

If the robot held the blue balls and waited for a better time to poop, the blue
balls would not cause the field to be jumbled.

To do this, I need to redesign the driver control scheme.

## Objective

To rework the intake code to allow for holding the balls during driver control,
allow for complex partner control, and simplify the code.

## Problem

Currently, the intake code is structured in way so that each combination of
buttons are represented by a state. This means `shoot` (rollers on, intake off)
is a different state than `on` (all on), even though they require the same logic
for autopooping, spaced shooting, ect. To add to the bloat, I needed non-pooping
variants of each state to be able to control during autonomous.

This structure does not scale well, and will be difficult to change to support
the new controls.

## Plan

I have an idea to greatly simplify the statemachine. Currently, there is a
unique enum (number) for each possible state.

For example, consider the states `on`, `shoot`, `onWithoutPoop`, and
`shootWithoutPoop`. If we break down what each action represents:

| State              | Intake | Bottom Roller | Top Roller | Poop |
| ------------------ | ------ | ------------- | ---------- | ---- |
| `on`               | yes    | yes           | yes        | yes  |
| `shoot`            | no     | yes           | yes        | yes  |
| `onWithoutPoop`    | yes    | yes           | yes        | no   |
| `shootWithoutPoop` | no     | yes           | yes        | no   |

We notice there is a lot of duplication

The idea is, what if we can represent the robot in the binary representation of
the state number?

For example, instead of `on = 1`, `shoot = 2`, etc, we have `on = 0b1111`,
`shoot = 0b0111`, `onWithoutPoop = 0b1110`, ect. This will let us "build" states
using bitwise logic, but still have each state named in an enum. There won't
need to be a LUT for the relation between state -> should intake, for example.
The code can directly measure whether something is enabled.

The best thing is, the state can both be treated as a distinct number, or a
collection of flags! This makes it really versatile in programming.

In short:

- Represent state as flag toggles
- Use bit manipulation with bitmasks
- Autopoop is off by default
- Name all the states to be switchable in the inner statemachine
- Assemble a state in driver control
- Have a bitrange where actions not supported by the flag schema can be run (ex.
  a number for deploy)

## New Controls

To start off, let's determine a new driver control scheme. Sawyer will be
responsible for shooting, intaking of red balls, and driving. I will be
responsible for dealing with the blue balls in the goals. I will have controls
which I can use to intake the blue balls, stop the intake, and then enable auto
auto poop.

**Sawyer**

- L2: outtake all (poop)
- R2: intake
- R1: shoot
- R1/R2: shoot and intake
- L2/R2: intake while reversing rollers
- L2/R1: outtake while shooting rollers
- L2/R1/R1: on

**Theo**

- R2: intake (no poop)
- R1: outtake (no poop)
- R1/R2: stop and hold balls

**Outtake behaviour**

- outtake = outtake everything
- outtake + intake = intake while reversing top = intake while sucking in failed
  shot, compacting (I think middle roller needs to be reversed)
- outtake + shoot = shoot while outtaking intakes = follow through with shot
  while outtaking, get rid of balls as fast as possible (I think middle roller
  should be reversed)
- outtake + shoot + intake = maybe just same as the last one except middle
  roller is normal direction

## Implementation

### Driver control

First, we can refactor the driver control code to provide the new functionality
and use the new api.

```cpp
// initial state
rollerStates state {rollerStates::off};

// if L2, add outtake
if (master(L2)) { state |= rollerStates::out; }

// if R2, add intake
if (master(R2)) { state |= rollerStates::intake; }

// partner controls
if (partner(R2) && partner(R1)) {
  state = rollerStates::off;
} else if (partner(R2) || partner(B)) {
  state = rollerStates::intake;
} else if (partner(R1)) {
  state = rollerStates::out;
} else if (master(RIGHT)) {
} else {
  // if none of the above buttons where pressed, then add poop
  state |= rollerStates::poop;
}

// partner can force poop
if (partner(B)) { state |= rollerStates::poop; }

// if R1, add shoot
if (master(R1)) { state |= rollerStates::shoot; }

// add actions
if (either(B)) {
  state |= rollerStates::deploy;
} else if (master(L1)) {
  state |= rollerStates::shootRev;
}

Robot::roller()->setNewState(state);
```

### State overview

Then, we can design a binary scheme for the states and enum (name) them all:

```cpp
// The first 3 bits toggle the state of each roller.
// The next few bits modify the behavior of the rollers.
// The last few bits enumerate additional actions.
// Bits can be combined to represent compound states.
enum class rollerStates {
  off = 0, // all off, no poop

  // toggle bits
  intake = 1 << 0, // move intakes
  bottom = 1 << 1, // move bottom roller
  top = 1 << 2, // move top roller

  // modifier bits
  out = 1 << 3, // reverse anything that is not enabled
  poop = 1 << 4, // enable auto poop

  // action bits
  deploy = 1 << 5,
  timedPoop = 2 << 5, // poop for time
  backPoop = 3 << 5, // bring blue down then poop
  shootRev = 4 << 5, // bring rollers back manually

  // state combinations
  loading = intake | bottom, // load balls into robot. Disable rollers one by one.
  shoot = bottom | top, // all on but don't move intakes
  on = intake | bottom | top, // all on

  compact = out | intake, // reverse top and botton rollers while intaking
  fill = out | shoot, // shoot top and bottom while outtaking
  purge = out | top, // shoot top while reversing rest

  loadingPoop = loading | poop,
  shootPoop = shoot | poop,
  onPoop = on | poop,
  outPoop = out | poop,

  toggles = 0b00000111, // bitmask for roller toggles
  modifiers = 0b00011000, // bitmask for roller modifiers
  flags = 0b00011111, // bitmask for flags
  actions = 0b11100000, // bitmask for actions
};
```

Finally, the internal statemachine implementation is similar to before, but
without a lot of redundant code:

```cpp
Roller::colors Roller::getTopLight() const {
  if (topLight->getProximity() < 100) return colors::none;

  int hue = topLight->getHue();
  switch (hue) {
    case 160 ... 300: return colors::blue;
    case 0 ... 40:
    case 350 ... 360: return colors::red;
    default: return colors::none;
  }
}

Roller::colors Roller::getBottomLight() const {
  if (bottomLight->getProximity() < 110) return colors::none;

  int hue = bottomLight->getHue();
  switch (hue) {
    case 180 ... 300: return colors::blue;
    case 0 ... 40:
    case 350 ... 360: return colors::red;
    default: return colors::none;
  }
}

void Roller::runAction(const rs& action) {
  // mark action start time
  macroTime.placeMark();
  // clear prev action
  state &= ~rs::actions;
  // add new action
  state |= action;
}

bool Roller::shouldPoop() {
  // if poop is not enabled, don't do anything
  if (!(state & rs::poop)) return false;

  // if a blue is in the top, lower it before pooping
  if (getTopLight() == colors::blue) {
    runAction(rs::backPoop);
    return true;
  }

  // if blue ball in bottom but no red in top, poop it
  if (getBottomLight() == colors::blue && getTopLight() != colors::red) {
    runAction(rs::timedPoop);
    return true;
  }

  return false;
}

int Roller::getIntake() {
  return !!(state & rs::intake) ? 12000 : (!!(state & rs::out) ? -12000 : 0);
}

void Roller::loop() {
  Rate rate;

  while (true) {

    // strip flags from action. If action, execute it, and skip flags.
    if (auto action = state & rs::actions; action != rs::off) {
      switch (action) {

        case rs::deploy:
          top(12000);
          bottom(-800);
          intake(-12000);
          break;

        case rs::timedPoop:
          top(-12000);
          bottom(12000);
          intake(getIntake());
          if (macroTime.getDtFromMark() >= 200_ms) {
            runAction(rs::off);
            continue;
          }
          break;

        case rs::backPoop:
          top(-12000);
          bottom(-12000);
          intake(getIntake());
          if (macroTime.getDtFromMark() >= 100_ms) {
            runAction(rs::off);
            continue;
          }
          break;

        case rs::shootRev:
          top(-12000);
          bottom(-12000);
          intake(getIntake());
          break;

        default:
          std::cout << "Error: action not found" << std::endl;
          break;
      }
      continue;
    }

    // run auto poop if needed
    if (shouldPoop()) continue;

    // if out is enabled
    bool out = !!(state & rs::out);

    // strip toggles from state
    switch (auto toggles = state & rs::toggles; toggles) {

      case rs::off:
        top(out ? -12000 : 0);
        bottom(out ? -12000 : 0);
        intake(out ? -12000 : 0);
        break;

      case rs::top:
        top(12000);
        bottom(out ? -12000 : 0);
        intake(out ? -12000 : 0);
        break;

      case rs::intake:
        if (out) {
          top(-12000);
          bottom(-12000);
        } else if (getTopLight() != colors::none && getBottomLight() != colors::none) {
          top(0);
          bottom(0);
        } else if (getTopLight() != colors::none) {
          // balance between raising ball to prevent rubbing and bringing ball too high
          top(0);
          bottom(6000);
        } else {
          // balance between bringing ball too fast and accidentally pooping
          top(7000);
          // if there is a blue but it can't poop
          bottom(12000);
        }
        intake(12000);
        break;

      case rs::shoot:
      case rs::on:
        // don't shoot a blue ball
        if (getTopLight() == colors::blue) {
          // move to intake mode
          state &= ~rs::shoot;
          top(-6000);
          pros::delay(100);
          continue;
        } else {
          top(12000);
        }
        // if red on the top needs to be separated from bottom ball
        if (getTopLight() == colors::red && getBottomLight() != colors::none) {
          if (getBottomLight() == colors::blue) {
            bottom(0);
          } else {
            bottom(2000);
          }
        } else {
          bottom(12000);
        }
        intake(getIntake());
        break;

      default:
        std::cout << "Error: toggle not found" << std::endl;
        break;
    }

    rate.delayUntil(5_ms);
  }
}
```

## Summary

In short, I took my statemachine to the next level by having the state enum
represent its state in its binary form.

With the binary scheme defined in the rollerStates, "shoot with autopoop" is
`0b00010110` and turning on the intakes would be flipping the first bit.

It's really cool because you can treat it like a normal enum/statemachine, each
state has a number, but you can also manipulate it as bitwise flags

So I can know which states have intake enabled, without needing a LUT or
hardcoding it, and the driver code is super small. The driver code uses the
bitwise representation of the state.

But the internal statemachine interprets the state as distinct states and
switches between the possibilities, allowing for more complex logic.

## Update March 28

I finished tuning up the code and tested it on the robot for the first time. It
worked perfectly first try! I'm looking forward to driver practice with the new
controls and automation.
