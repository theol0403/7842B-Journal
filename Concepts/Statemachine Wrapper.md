# Statemachine Wrapper

A statemachine is an elegant way of programming a subsystem, and makes it easy to control both in driver control and autonomous. A statemachine is usually implemented using an enum containing a list of states, and then a switch statement in a separate thread that implements each state.

However, implementing a statemachine requires a substantial amount of boilerplate, both to set up the task but also to set up the setter and getter functions. This is why I wrote an abstract `StateMachine` wrapper that reduces said boilerplate. `StateMachine` inherits task functionality from [Task Wrapper](Task Wrapper).

Here is the class, which accepts the enum type as a template paramiter:

```cpp
/**
 * State machine helper class.
 *
 * @tparam States       An enum class representing the states of a subsystem. Required to have an
 *                      off state.
 * @tparam assumedState Optional - The assumed last state when using setNewState. Initially calling
 *                      setNewState with this state will not trigger a state transition.
 */
template <typename States, States assumedState = States::off>
class StateMachine : public TaskWrapper {

 public:
  StateMachine() = default;
  virtual ~StateMachine() = default;

  /**
   * Sets the state.
   *
   * @param istate The state
   */
  virtual void setState(const States& istate) {
    state = istate;
  }

  /**
   * Sets the state only if the state is different from the last time this function was called.
   *
   * @param istate The state
   */
  virtual void setNewState(const States& istate) {
    if (istate != lastState) {
      state = istate;
      lastState = istate;
    }
  }

  /**
   * Gets the state.
   *
   * @return The state.
   */
  virtual const States& getState() const {
    return state;
  }

 protected:
  /**
   * Override this method to implement setup procedures.
   */
  virtual void calibrate() = 0;

  /**
   * Override this method to implement the statemachine task
   */
  void loop() override = 0;

  States state {States::off};
  States lastState {assumedState};
};
```

