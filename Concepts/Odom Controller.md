# Odom Controller

Here is the API (header file) for `OdomController`, which belongs to lib7842.

```cpp
/**
 * Function that returns true to end chassis movement
 * Used to implement different settling methods
 */
using Settler = std::function<bool(const OdomController& odom)>;

/**
 * Function that accepts a turning velocity and controls excecution to the chassis
 * Used to implement a point or pivot turn
 */
using Turner = std::function<void(ChassisModel& model, double vel)>;

/**
 * Function that returns an angle for the chassis to seek
 * Examples can be an AngleCalculator that returns the angle to a point, or an angle to an absolute angle
 */
using AngleCalculator = std::function<QAngle(const OdomController& odom)>;

class OdomController {

public:
  /**
   * OdomController. Implements chassis movement algorithms
   *
   * @param imodel              The chassis model
   * @param iodometry           The chassis odometry
   * @param idistanceController The distance pid controller
   * @param iturnController     The turning pid controller
   * @param iangleController    The angle pid controller, used to keep distance driving straight
   * @param isettleRadius       The radius from the target point to give up angle correction
   */
  OdomController(
    const std::shared_ptr<ChassisModel>& imodel,
    const std::shared_ptr<CustomOdometry>& iodometry,
    std::unique_ptr<IterativePosPIDController> idistanceController,
    std::unique_ptr<IterativePosPIDController> iturnController,
    std::unique_ptr<IterativePosPIDController> iangleController,
    const QLength& isettleRadius);

  virtual ~OdomController() = default;

  /**
   * Turn the chassis using the given AngleCalculator
   *
   * @param angleCalculator The angle calculator
   * @param turner          The turner
   * @param settler         The settler
   */
  virtual void turn(
    const AngleCalculator& angleCalculator,
    const Turner& turner = pointTurn,
    const Settler& settler = defaultTurnSettler);

  /**
   * Turn the chassis to face an absolue angle
   *
   * @param angle   The angle
   * @param turner  The turner
   * @param settler The settler
   */
  virtual void turnToAngle(
    const QAngle& angle, const Turner& turner = pointTurn, const Settler& settler = defaultTurnSettler);

  /**
   * Turn the chassis to face a relative angle
   *
   * @param angle   The angle
   * @param turner  The turner
   * @param settler The settler
   */
  virtual void turnAngle(
    const QAngle& angle, const Turner& turner = pointTurn, const Settler& settler = defaultTurnSettler);

  /**
   * Turn the chassis to face a point
   *
   * @param point   The point
   * @param turner  The turner
   * @param settler The settler
   */
  virtual void turnToPoint(
    const Vector& point, const Turner& turner = pointTurn, const Settler& settler = defaultTurnSettler);

  /**
   * Drive a distance while correcting angle using an AngleCalculator 
   *
   * @param distance        The distance
   * @param angleCalculator The angle calculator
   * @param turnScale       The turn scale
   * @param settler         The settler
   */
  virtual void moveDistanceAtAngle(
    const QLength& distance,
    const AngleCalculator& angleCalculator,
    double turnScale,
    const Settler& settler = defaultDriveSettler);

  /**
   * Drive a distance while maintaining starting angle
   *
   * @param distance The distance
   * @param settler  The settler
   */
  virtual void moveDistance(const QLength& distance, const Settler& settler = defaultDriveSettler);

  /**
   * Drive to a point using custom point seeking
   *
   * @param targetPoint The target point
   * @param turnScale   The turn scale used to control the priority of turning over driving. A higher value will make
   *                    the robot turn to face the point sooner
   * @param settler     The settler
   */
  virtual void driveToPoint(
    const Vector& targetPoint, double turnScale = 1, const Settler& settler = defaultDriveSettler);

  /**
   * Drive to a point using simple point seeking
   *
   * @param targetPoint The target point
   * @param turnScale   The turn scale used to control the priority of turning over driving. A higher value will make
   *                    the robot turn to face the point sooner
   * @param settler     The settler
   */
  virtual void driveToPoint2(
    const Vector& targetPoint, double turnScale = 1, const Settler& settler = defaultDriveSettler);

  /**
   * A Settler that is used for turning which uses the turning pid's isSettled() method
   */
  static bool defaultTurnSettler(const OdomController& odom);

  /**
   * A Settler that is used for driving which uses the distance pid's isSettled() method
   */
  static bool defaultDriveSettler(const OdomController& odom);

  /**
   * A Settler that is used for driving which uses the distance and angle pid's isSettled() method
   */
  static bool defaultDriveAngleSettler(const OdomController& odom);

  /**
   * A Turner that excecutes a point turn which turns in place. Used as default for turn functions
   */
  static void pointTurn(ChassisModel& model, double vel);

  /**
   * A Turner that excecutes a left pivot, meaning it only moves the left motors.
   */
  static void leftPivot(ChassisModel& model, double vel);

  /**
   * A Turner that excecutes a right pivot, meaning it only moves the right motors.
   */
  static void rightPivot(ChassisModel& model, double vel);

  /**
   * Make a Settler that exits when angle error is within given range
   * @param angle The angle error theshold
   */
  static Settler makeSettler(const QAngle& angle);

  /**
   * Make a Settler that exits when distance error is within given range
   * @param distance The distance error theshold
   */
  static Settler makeSettler(const QLength& distance);

  /**
   * Make a Settler that exits when both angle and distance error is within given range.
   * @param angle The angle error theshold
   * @param distance The distance error theshold
   */
  static Settler makeSettler(const QLength& distance, const QAngle& angle);

  /**
   * Generates an AngleCalculator that seeks a given absolute angle
   *
   * @param angle The angle
   */
  static AngleCalculator makeAngleCalculator(const QAngle& angle);

  /**
   * Generates an AngleCaclulator that seeks a given point.
   *
   * @param point The point
   */
  static AngleCalculator makeAngleCalculator(const Vector& point);

  /**
   * Generates an AngleCaclulator that returns a constant error.
   *
   * @param error The error
   */
  static AngleCalculator makeAngleCalculator(double error);

  /**
   * Generates an AngleCalculator that does nothing
   */
  static AngleCalculator makeAngleCalculator();

  /**
   * Calculates angle from the chassis to the point
   */
  QAngle angleToPoint(const Vector& point) const;

  /**
   * Calculates distance from the chassis to the point
   */
  QLength distanceToPoint(const Vector& point) const;

protected:
  /**
   * Resets the pid controllers, used before every motion
   */
  virtual void resetPid();

  /**
   * Controls the chassis movement
   * Applies magnitude control to prioritize turning
   *
   * @param forwardSpeed Forward speed
   * @param yaw          The yaw
   */
  void driveVector(double forwardSpeed, double yaw);

  std::shared_ptr<ChassisModel> model {nullptr};
  std::shared_ptr<CustomOdometry> odometry {nullptr};
  std::unique_ptr<IterativePosPIDController> distanceController {nullptr};
  std::unique_ptr<IterativePosPIDController> angleController {nullptr};
  std::unique_ptr<IterativePosPIDController> turnController {nullptr};
  const QLength settleRadius;

  QAngle angleErr = 0_deg;
  QLength distanceErr = 0_in;
};
```

# Odom X Controller

Here is the API for `OdomXController`, which implements strafing algorithms:

```cpp
class OdomXController : public OdomController {

public:
  /**
   * OdomXController. Implements chassis movement algorithms for the X drive.
   *
   * @param imodel              The chassis model
   * @param iodometry           The chassis odometry
   * @param idistanceController The distance pid controller
   * @param iturnController     The turning pid controller
   * @param iangleController    The angle pid controller, used to keep distance driving straight
   * @param istrafeController   The strafe pid controller
   * @param isettleRadius       The radius from the target point to give up angle correction
   */
  OdomXController(
    const std::shared_ptr<XDriveModel>& imodel,
    const std::shared_ptr<CustomOdometry>& iodometry,
    std::unique_ptr<IterativePosPIDController> idistanceController,
    std::unique_ptr<IterativePosPIDController> iturnController,
    std::unique_ptr<IterativePosPIDController> iangleController,
    std::unique_ptr<IterativePosPIDController> istrafeController,
    const QLength& isettleRadius);

  virtual ~OdomXController() = default;

  /**
   * Strafe a distance in a relative direction while correcting angle using an AngleCalculator
   *
   * @param distance        The distance
   * @param direction       The relative direction of the strafing
   * @param angleCalculator The angle calculator
   * @param turnScale       The turn scale
   * @param settler         The settler
   */
  virtual void strafeDistance(
    const QLength& distance,
    const QAngle& direction,
    const AngleCalculator& angleCalculator = makeAngleCalculator(),
    double turnScale = 1,
    const Settler& settler = defaultDriveSettler);

  /**
   * Strafe a distance in an absolute direction while correcting angle using an AngleCalculator
   *
   * @param distance        The distance
   * @param direction       The absolute direction of the strafing
   * @param angleCalculator The angle calculator
   * @param turnScale       The turn scale
   * @param settler         The settler
   */
  virtual void strafeDistanceAtDirection(
    const QLength& distance,
    const QAngle& direction,
    const AngleCalculator& angleCalculator = makeAngleCalculator(),
    double turnScale = 1,
    const Settler& settler = defaultDriveSettler);

  /**
   * Drive to a point using custom point seeking for strafing and an AngleCalculator
   *
   * @param targetPoint     The target point
   * @param angleCalculator The angle calculator
   * @param turnScale       The turn scale used to control the priority of turning over driving. A higher value will
   *                        make the robot turn to face the point sooner
   * @param settler         The settler
   */
  virtual void driveToPoint(
    const Vector& targetPoint,
    const AngleCalculator& angleCalculator = makeAngleCalculator(),
    double turnScale = 1,
    const Settler& settler = defaultStrafeSettler);

  /**
   * Drive to a point using custom point seeking
   *
   * @param targetPoint The target point
   * @param turnScale   The turn scale used to control the priority of turning over driving. A higher value will make
   *                    the robot turn to face the point sooner
   * @param settler     The settler
   */
  void driveToPoint(
    const Vector& targetPoint,
    double turnScale = 1,
    const Settler& settler = defaultStrafeSettler) override;

  /**
   * Drive to a point using custom point seeking for strafing and an AngleCalculator
   *
   * @param targetPoint     The target point
   * @param angleCalculator The angle calculator
   * @param settler         The settler
   */
  virtual void strafeToPoint(
    const Vector& targetPoint,
    const AngleCalculator& angleCalculator = makeAngleCalculator(),
    double turnScale = 1,
    const Settler& settler = defaultDriveSettler);

  /**
   * A Settler that is used for driving/strafing which uses the distance and strafe pid's isSettled() method
   */
  static bool defaultStrafeSettler(const OdomController& odom);

protected:
  /**
   * Resets the pid controllers, used before every motion
   */
  void resetPid() override;

  /**
   * Controls the chassis movement. Applies magnitude control to prioritize turning.
   *
   * @param forwardSpeed The forward speed
   * @param yaw          The yaw speed
   * @param strafe       The strafe speed
   */
  void driveXVector(double forwardSpeed, double yaw, double strafe);

  /**
   * Controls the chassis movement. Strafes at the given speed at the given direction.
   *
   * @param speed     The speed
   * @param direction The direction
   * @param yaw       The yaw
   */
  void strafeXVector(double speed, const QAngle& direction, double yaw);

  std::shared_ptr<XDriveModel> model {nullptr};
  std::unique_ptr<IterativePosPIDController> strafeController {nullptr};
};
```
