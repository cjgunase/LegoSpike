# Lesson 02: 20 LEGO SPIKE Prime Project Blueprints

Each blueprint below is structured with a quick build guide, core robotics concept, pseudocode, and Python sample (for the SPIKE Prime Python runtime). Builds assume a standard two-motor drivetrain on ports C (left) and D (right) unless noted, plus sensors as listed.

## 1. Line Follower Lab
- **Build**: Two-wheel drive, color sensor angled ~1 cm above floor between wheels. Route black tape line on light surface.
- **Concepts**: Feedback loop, proportional control.
- **Pseudocode**:
  ```text
  set motors C+D
  set target_reflect = 50
  set kp = 0.8
  loop forever
    error = target_reflect - color.reflected_light()
    turn = kp * error
    set left speed = base - turn
    set right speed = base + turn
  ```
- **Python**:
  ```python
  from spike import PrimeHub, ColorSensor, MotorPair

  hub = PrimeHub()
  drive = MotorPair('C', 'D')
  color = ColorSensor('E')

  target = 50
  kp = 0.8
  base = 40

  while True:
    error = target - color.get_reflected_light()
    turn = kp * error
    drive.start_tank(base - turn, base + turn)
  ```

## 2. Wall-Stop Rover
- **Build**: Distance sensor front-facing ~4–6 cm above ground.
- **Concepts**: Thresholds, safe stop.
- **Pseudocode**:
  ```text
  start driving forward at 40%
  wait until distance < 15 cm
  brake then stop
  ```
- **Python**:
  ```python
  from spike import PrimeHub, DistanceSensor, MotorPair

  hub = PrimeHub()
  dist = DistanceSensor('F')
  drive = MotorPair('C', 'D')

  drive.start(40)
  while dist.get_distance_cm() is None or dist.get_distance_cm() > 15:
    pass
  drive.stop()
  ```

## 3. Precision Parking Bot
- **Build**: Standard drive. Mark a parking spot with tape.
- **Concepts**: Dead-reckoning, rotations to distance.
- **Pseudocode**:
  ```text
  reset motor rotations
  target_rotations = 2.0
  start forward at 40%
  wait until average_rotation >= target_rotations
  stop, reverse 0.5 rotations
  ```
- **Python**:
  ```python
  from spike import Motor, MotorPair

  left = Motor('C')
  right = Motor('D')
  drive = MotorPair('C', 'D')

  left.set_degrees_counted(0)
  right.set_degrees_counted(0)
  drive.start(40)
  while (left.get_degrees_counted() + right.get_degrees_counted())/720 < 2.0:
    pass
  drive.stop()
  drive.move_tank(-0.5, 'rotations', -30, -30)
  ```

## 4. Maze Navigator
- **Build**: Distance sensor forward, color sensor downward to detect intersections.
- **Concepts**: State machine, branching logic.
- **Pseudocode**:
  ```text
  while not at goal color (green)
    if distance < 15: turn right 90
    else if color sees blue: turn left 90
    else drive forward
  stop
  ```
- **Python**:
  ```python
  from spike import DistanceSensor, ColorSensor, MotorPair

  dist = DistanceSensor('F')
  color = ColorSensor('E')
  drive = MotorPair('C', 'D')

  while color.get_color() != 'green':
    d = dist.get_distance_cm()
    if d is not None and d < 15:
      drive.move_tank(0.25, 'rotations', 30, -30)
    elif color.get_color() == 'blue':
      drive.move_tank(0.25, 'rotations', -30, 30)
    else:
      drive.start(30)
  drive.stop()
  ```

## 5. Delivery Drone on Wheels
- **Build**: Attach small hinged arm on a third motor at port B to drop package.
- **Concepts**: Sequencing, functions.
- **Pseudocode**:
  ```text
  function drop_package(): rotate arm 90 deg, pause, reset
  drive to waypoint A (rotations)
  drop_package()
  drive back to start
  ```
- **Python**:
  ```python
  from spike import Motor, MotorPair

  arm = Motor('B')
  drive = MotorPair('C', 'D')

  def drop_package():
    arm.run_for_degrees(90, 30)
    arm.run_for_degrees(-90, 30)

  drive.move(30, 'cm', 40)
  drop_package()
  drive.move(-30, 'cm', 40)
  ```

## 6. Factory Conveyor Tester
- **Build**: Use a small conveyor (belt or rollers) on motor B; color sensor above belt; sorter flap on motor A.
- **Concepts**: Event-driven sorting.
- **Pseudocode**:
  ```text
  start conveyor
  when color is red -> move flap left
  when color is blue -> move flap right
  default -> center
  ```
- **Python**:
  ```python
  from spike import Motor, ColorSensor
  import time

  conveyor = Motor('B')
  flap = Motor('A')
  color = ColorSensor('E')

  conveyor.start(40)
  while True:
    c = color.get_color()
    if c == 'red':
      flap.run_to_position(90, 'clockwise')
    elif c == 'blue':
      flap.run_to_position(-90, 'counterclockwise')
    else:
      flap.run_to_position(0, 'shortest path')
    time.sleep(0.1)
  ```

## 7. Traffic Light Intersection
- **Build**: Light tower using hub LEDs; small car robot with distance sensor to react.
- **Concepts**: Timing cycles, parallel logic.
- **Pseudocode**:
  ```text
  loop lights: green 4s -> yellow 1s -> red 4s
  car drives until sees red; stops; goes on green
  ```
- **Python** (single hub running both behaviors):
  ```python
  from spike import PrimeHub, DistanceSensor, MotorPair
  import time

  hub = PrimeHub()
  dist = DistanceSensor('F')
  drive = MotorPair('C', 'D')

  def cycle_lights():
    hub.light_matrix.show_image('ARROW_N')
    time.sleep(4)
    hub.light_matrix.show_image('HEART')
    time.sleep(1)
    hub.light_matrix.show_image('NO')
    time.sleep(4)

  while True:
    cycle_lights()
    if hub.light_matrix.get_pixels() == hub.light_matrix.NO:
      drive.stop()
    else:
      drive.start(40)
  ```

## 8. Dance Choreography Bot
- **Build**: Standard drive; optional speaker via hub.
- **Concepts**: Timed sequencing, rhythm.
- **Pseudocode**:
  ```text
  steps = list of (move, duration, light)
  for step in steps
    set light
    perform move (turn/drive)
    wait duration
  stop
  ```
- **Python**:
  ```python
  from spike import PrimeHub, MotorPair
  import time

  hub = PrimeHub()
  drive = MotorPair('C', 'D')

  routine = [
    ('forward', 1.0, 'blue'),
    ('spin', 0.7, 'yellow'),
    ('back', 1.0, 'purple')
  ]

  for move, duration, color in routine:
    hub.light_matrix.set_pixels([[9]])
    if move == 'forward':
      drive.start(40)
    elif move == 'back':
      drive.start(-40)
    elif move == 'spin':
      drive.start_tank(40, -40)
    time.sleep(duration)
    drive.stop()
  ```

## 9. Sumo Pusher
- **Build**: Low bumper up front; color sensor downward to detect ring edge (black tape); distance sensor to spot opponent.
- **Concepts**: Priority logic, safety check.
- **Pseudocode**:
  ```text
  loop
    if edge detected -> reverse and turn
    else if opponent close -> charge forward
    else spin slowly to search
  ```
- **Python**:
  ```python
  from spike import DistanceSensor, ColorSensor, MotorPair

  dist = DistanceSensor('F')
  color = ColorSensor('E')
  drive = MotorPair('C', 'D')

  while True:
    if color.get_color() == 'black':
      drive.move_tank(-0.5, 'rotations', -40, 40)
    elif dist.get_distance_cm() is not None and dist.get_distance_cm() < 20:
      drive.start(70)
    else:
      drive.start_tank(20, -20)
  ```

## 10. Smart Pet Feeder
- **Build**: Small hopper with motor B to rotate a gate; color or force sensor to confirm drop.
- **Concepts**: Scheduling, verification.
- **Pseudocode**:
  ```text
  every 30 minutes: run motor 180 deg
  if sensor confirms object -> success else alert light
  ```
- **Python**:
  ```python
  from spike import PrimeHub, Motor, ColorSensor
  import time

  hub = PrimeHub()
  gate = Motor('B')
  color = ColorSensor('E')

  while True:
    gate.run_for_degrees(180, 40)
    if color.get_reflected_light() < 20:
      hub.light_matrix.show_image('HAPPY')
    else:
      hub.light_matrix.show_image('SAD')
    time.sleep(1800)
  ```

## 11. Bridge Balance Tester
- **Build**: Gyro/IMU in hub; drive slowly across test bridge.
- **Concepts**: Tilt threshold, safe stop.
- **Pseudocode**:
  ```text
  drive forward slowly
  if pitch angle > 15 deg -> stop and back up
  ```
- **Python**:
  ```python
  from spike import PrimeHub, MotorPair
  import time

  hub = PrimeHub()
  drive = MotorPair('C', 'D')

  drive.start(20)
  while True:
    if abs(hub.motion_sensor.get_pitch_angle()) > 15:
      drive.stop()
      drive.move(-20, 'cm', 30)
      break
    time.sleep(0.05)
  ```

## 12. Greenhouse Vent Controller
- **Build**: Vent flap on motor B; temperature simulated via variable or external input (button cycles modes).
- **Concepts**: Hysteresis, actuation.
- **Pseudocode**:
  ```text
  temp_high = 28C, temp_low = 24C
  if temp > temp_high -> open vent
  if temp < temp_low -> close vent
  ```
- **Python**:
  ```python
  from spike import Motor

  vent = Motor('B')
  temp = 25
  temp_high, temp_low = 28, 24

  while True:
    # replace with real sensor if available
    if temp > temp_high:
      vent.run_to_position(90, 'clockwise')
    elif temp < temp_low:
      vent.run_to_position(0, 'shortest path')
    # adjust temp manually in code or via buttons
  ```

## 13. Stepper Plotter
- **Build**: Two-wheel drive with marker mounted centrally; grid mat on floor.
- **Concepts**: Coordinate thinking, functions.
- **Pseudocode**:
  ```text
  function forward_grid(n): move n squares
  function turn_right(): 90 deg spin
  draw square: forward 2, right, forward 2, right ...
  ```
- **Python**:
  ```python
  from spike import MotorPair

  drive = MotorPair('C', 'D')

  def forward_grid(n):
    drive.move(n * 20, 'cm', 30)

  def turn_right():
    drive.move_tank(0.25, 'rotations', 30, -30)

  for _ in range(4):
    forward_grid(2)
    turn_right()
  ```

## 14. Elevator Simulator
- **Build**: Vertical track with platform on string driven by motor B; buttons (hub buttons) select floors.
- **Concepts**: Queues, state machine.
- **Pseudocode**:
  ```text
  floors = [0,1,2]
  current = 0
  when button pressed -> target floor
  move motor to height per floor (degrees)
  ```
- **Python**:
  ```python
  from spike import PrimeHub, Motor
  import time

  hub = PrimeHub()
  lift = Motor('B')
  floor_degrees = {0:0, 1:180, 2:360}
  current = 0

  while True:
    if hub.left_button.is_pressed():
      target = 1
    elif hub.right_button.is_pressed():
      target = 2
    else:
      target = 0
    lift.run_to_position(floor_degrees[target], 'shortest path')
    current = target
    time.sleep(0.2)
  ```

## 15. Treasure Guard
- **Build**: Distance sensor forward; LEDs for alarm; optional buzzer (hub sound).
- **Concepts**: Patrol loop, alert state.
- **Pseudocode**:
  ```text
  patrol: forward 10 cm, turn 90, repeat
  if distance < 20 -> alarm lights and stop
  ```
- **Python**:
  ```python
  from spike import PrimeHub, DistanceSensor, MotorPair
  import time

  hub = PrimeHub()
  dist = DistanceSensor('F')
  drive = MotorPair('C', 'D')

  while True:
    drive.move(10, 'cm', 30)
    drive.move_tank(0.25, 'rotations', 30, -30)
    if dist.get_distance_cm() is not None and dist.get_distance_cm() < 20:
      hub.light_matrix.show_image('ANGRY')
      drive.stop()
      break
    time.sleep(0.1)
  ```

## 16. Adaptive Cruise Control
- **Build**: Distance sensor forward.
- **Concepts**: Proportional speed control, smoothing.
- **Pseudocode**:
  ```text
  target_gap = 25 cm
  speed = (distance - target_gap) * kp
  limit speed between 0 and 70
  ```
- **Python**:
  ```python
  from spike import DistanceSensor, MotorPair

  dist = DistanceSensor('F')
  drive = MotorPair('C', 'D')
  target = 25
  kp = 2

  while True:
    d = dist.get_distance_cm()
    if d is None:
      drive.stop()
      continue
    speed = max(0, min(70, (d - target) * kp))
    drive.start(speed)
  ```

## 17. Color-Code Mission Runner
- **Build**: Color cards on floor; color sensor downward.
- **Concepts**: Function mapping, data-driven design.
- **Pseudocode**:
  ```text
  map color -> action (turn, forward, drop)
  read card sequence; run mapped actions
  ```
- **Python**:
  ```python
  from spike import ColorSensor, MotorPair

  color = ColorSensor('E')
  drive = MotorPair('C', 'D')

  def forward(): drive.move(20, 'cm', 40)
  def left(): drive.move_tank(0.25, 'rotations', -30, 30)
  def right(): drive.move_tank(0.25, 'rotations', 30, -30)

  actions = {'red': forward, 'blue': left, 'yellow': right}

  for _ in range(3):
    c = color.get_color()
    action = actions.get(c, drive.stop)
    action()
  ```

## 18. Automated Gate
- **Build**: Barrier arm on motor B; distance sensor faces incoming car (object).
- **Concepts**: Event trigger, interlock.
- **Pseudocode**:
  ```text
  if distance < 20 -> raise arm, wait 3s, lower arm
  ignore triggers while arm moving
  ```
- **Python**:
  ```python
  from spike import DistanceSensor, Motor
  import time

  dist = DistanceSensor('F')
  arm = Motor('B')

  busy = False
  while True:
    d = dist.get_distance_cm()
    if not busy and d is not None and d < 20:
      busy = True
      arm.run_to_position(90, 'clockwise')
      time.sleep(3)
      arm.run_to_position(0, 'shortest path')
      busy = False
    time.sleep(0.1)
  ```

## 19. Robot Waiter
- **Build**: Tray platform; distance sensor front; buttons select table A/B.
- **Concepts**: Waypoints, task decomposition.
- **Pseudocode**:
  ```text
  if left button -> path A (forward, left turn)
  if right button -> path B (forward, right turn)
  stop if obstacle < 15 cm
  ```
- **Python**:
  ```python
  from spike import PrimeHub, DistanceSensor, MotorPair
  import time

  hub = PrimeHub()
  dist = DistanceSensor('F')
  drive = MotorPair('C', 'D')

  def safe_move(cm):
    drive.start(40)
    moved = 0
    while moved < cm:
      if dist.get_distance_cm() is not None and dist.get_distance_cm() < 15:
        drive.stop()
        return False
      time.sleep(0.05)
      moved += 2
    drive.stop()
    return True

  while True:
    if hub.left_button.is_pressed():
      safe_move(30)
      drive.move_tank(0.25, 'rotations', -30, 30)
    elif hub.right_button.is_pressed():
      safe_move(30)
      drive.move_tank(0.25, 'rotations', 30, -30)
  ```

## 20. Self-Diagnosing Bot
- **Build**: Standard drive; color sensor for feedback; hub LEDs for status.
- **Concepts**: Pre-mission checks, guard clauses.
- **Pseudocode**:
  ```text
  test motors: spin 0.25 rotations -> check running
  test color sensor: read light > threshold
  if any fail -> show red X; else green check and start mission
  ```
- **Python**:
  ```python
  from spike import PrimeHub, Motor, ColorSensor, MotorPair
  import time

  hub = PrimeHub()
  left = Motor('C')
  right = Motor('D')
  color = ColorSensor('E')
  drive = MotorPair('C', 'D')

  def motor_test():
    left.run_for_degrees(90, 20)
    right.run_for_degrees(90, 20)
    return True

  def color_test():
    return color.get_reflected_light() is not None

  if motor_test() and color_test():
    hub.light_matrix.show_image('HAPPY')
    drive.move(20, 'cm', 30)
  else:
    hub.light_matrix.show_image('SAD')
  ```
