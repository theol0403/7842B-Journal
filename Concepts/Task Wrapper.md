# Task Wrapper

If a class requires a task as a member, it causes some problems. A `pros::Task` requires a callback function to run, but due to some limitations in c++, the address of that function needs to be known in *compile-time*. This means that the task callback needs to be `static` , which means that it does not belong to the class instance and it can't access class members. To fix this, a pattern named **trampoline** is used. Trampoline is the act of passing a opaque pointer to the class object *through the task* to be received by the static function, having the static function cast the pointer to the correct class type, and calling a member function to execute the task.

Since implementing a trampoline requires a solid amount of boilerplate, I have written an abstract task wrapper that does this for me. To be able to run using unit tests and CI, I am using OkapiLib's `CrossPlatformTask` as the task object.

Note how the `this` pointer is passed when task is constructed:

```cpp
task = std::make_unique<CrossplatformThread>(trampoline, this, iname.c_str());
```

Then, the pointer is cast by the trampoline and a virtual member `loop()` is called, which is then resolved by dynamic binding:

```cpp
void TaskWrapper::trampoline(void* iparam) {
  pros::delay(20);
  static_cast<TaskWrapper*>(iparam)->loop();
}
```

Here is the full `TaskWrapper` implementation:

```cpp
class TaskWrapper {
protected:
  explicit TaskWrapper(const std::shared_ptr<Logger>& ilogger = Logger::getDefaultLogger());
  virtual ~TaskWrapper() = default;

  /**
   * Extend this function to implement custom task.
   */
  virtual void loop();

public:
  /**
   * Starts the task.
   *
   * @param iname The task name
   */
  virtual void startTask(const std::string& iname = "TaskWrapper");

  /**
   * Kills the task.
   */
  virtual void killTask();

  /**
   * Gets the task name.
   */
  virtual std::string getName();

protected:
  std::shared_ptr<Logger> logger {nullptr};

private:
  static void trampoline(void* iparam);
  std::unique_ptr<CrossplatformThread> task {nullptr};
};
```

```cpp
TaskWrapper::TaskWrapper(const std::shared_ptr<Logger>& ilogger) : logger(ilogger) {}

void TaskWrapper::loop() {
  std::string msg("TaskWrapper::loop: loop is not overridden");
  LOG_ERROR(msg);
  throw std::runtime_error(msg);
}

void TaskWrapper::startTask(const std::string& iname) {
  if (task) LOG_INFO("TaskWrapper::startTask: restarting task: " + iname);
  task = std::make_unique<CrossplatformThread>(trampoline, this, iname.c_str());
}

void TaskWrapper::killTask() {
  task = nullptr;
}

std::string TaskWrapper::getName() {
  return task->getName();
};

void TaskWrapper::trampoline(void* iparam) {
  pros::delay(20);
  static_cast<TaskWrapper*>(iparam)->loop();
}
```

