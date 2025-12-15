# Lesson 01.1: SPIKE Prime Python Basics

This companion lesson focuses purely on Python implementations for the LEGO SPIKE Prime kit. Each section introduces a small, runnable snippet you can try directly in the SPIKE app using the PrimeHub and common attachments (driving base with motors on ports C and D, Color Sensor on E, Distance Sensor on F, and Force Sensor on A). Adapt the port letters to match your build.

Reference docs: https://tuftsceeo.github.io/SPIKEPythonDocs/SPIKE3.html

## Quick Setup

```python
from spike import PrimeHub, Motor, MotorPair, ColorSensor, DistanceSensor, ForceSensor

hub = PrimeHub()
drive = MotorPair('C', 'D')
left_wheel = Motor('C')
right_wheel = Motor('D')
color = ColorSensor('E')
distance = DistanceSensor('F')
force = ForceSensor('A')
```

> Tip: If you have only two motors connected, you can still instantiate the sensors you need. The hub will ignore unplugged devices until you connect them.

## Motor Pair Driving (C + D)

These exercises use `MotorPair` to move a basic driving base.

1. **Drive a Straight Line** – set a consistent speed and stop after a distance.

```python
drive.set_default_speed(40)          # percent
# run 2 wheel-rotations worth of distance
# each rotation equals the wheel circumference (about 17.6 cm for the stock wheel)
drive.move(amount=2, unit='rotations')
```

2. **Precise Turn Using Steering** – spin left, then brake.

```python
drive.start(steering=-100, speed=35)
hub.wait_for_seconds(0.8)
drive.stop()
```

3. **Arc Turn** – combine forward motion with gentle steering.

```python
drive.move(20, unit='cm', steering=30)  # slight right arc for 20 cm
```

4. **Smooth Acceleration and Coasting Stop** – ramp up speed and let the built-in hold mode slow you down.

```python
for speed in range(20, 81, 15):
    drive.start(speed=speed)
    hub.wait_for_seconds(0.6)
drive.stop()
```

5. **Square Path Challenge** – chain four identical forward-and-turn steps.

```python
quarter_turn = 90

def drive_square(side_cm=25):
    for _ in range(4):
        drive.move(side_cm, unit='cm', speed=50)
        drive.move(quarter_turn, unit='degrees', steering=-100)  # ~90° spin

drive_square()
```

## Single Motor Control (C or D)

Use `Motor` for direct control of one wheel or an accessory (e.g., arm or grabber).

1. **Spin a Single Motor by Angle**

```python
left_wheel.run_for_degrees(360, speed=70)  # one full turn
```

2. **Run Until Stalled** – stop when the motor meets resistance (great for closing a claw gently).

```python
right_wheel.run_until_stalled(speed=40, stall_detection=True)
```

3. **Measure and Reset Position**

```python
left_wheel.set_degrees_counted(0)
left_wheel.run_for_rotations(1.5, speed=60)
print('Left wheel degrees:', left_wheel.get_degrees_counted())
```

## Color Sensor Basics (Port E)

1. **Read Color Names** – valid names include `red`, `blue`, `green`, `yellow`, `black`, `white`.

```python
color_name = color.get_color()
if color_name:
    hub.status_light.on(color_name)
    print('Saw:', color_name)
```

2. **Reflected Light for Line Following** – use grayscale to chase a line edge.

```python
target = 40  # pick based on your mat; 0=dark, 100=bright
kp = 0.8
while True:
    reflection = color.get_reflected_light()
    error = target - reflection
    correction = kp * error
    drive.start(steering=correction, speed=30)
```

3. **Ambient Light Reaction** – dim the hub lights in a dark room.

```python
level = color.get_ambient_light()
hub.light_matrix.write(str(level))
if level < 20:
    hub.status_light.on('blue')
```

## Distance Sensor Basics (Port F)

1. **Drive Until Close, Then Stop**

```python
drive.start(speed=40)
while distance.get_distance_cm() is None or distance.get_distance_cm() > 20:
    hub.wait_for_seconds(0.05)
drive.stop()
```

2. **Back Away From Obstacles** – reverse if something approaches.

```python
while True:
    reading = distance.get_distance_cm()
    if reading and reading < 15:
        drive.move(-10, unit='cm', speed=40)
    hub.wait_for_seconds(0.1)
```

3. **Gesture Start** – wave your hand to begin a routine.

```python
hub.light_matrix.show_image('HAPPY')
distance.wait_for_distance_closer_than(10, 'cm')
hub.light_matrix.show_image('ARROW_N')
drive.move(30, unit='cm', speed=60)
```

## Force Sensor Basics (Port A)

1. **Button-Style Start**

```python
hub.light_matrix.show_image('SQUARE')
force.wait_until_pressed()
hub.light_matrix.show_image('ARROW_N')
drive.move(25, unit='cm', speed=50)
```

2. **Measure Force for Proportional Control** – press harder to go faster.

```python
while True:
    applied = force.get_force_newton()  # roughly 0–10 N by hand
    speed = min(100, int(applied * 10))
    drive.start(speed=speed)
    hub.wait_for_seconds(0.1)
```

3. **Detect a Release to Stop**

```python
drive.start(speed=35)
force.wait_until_released()
drive.stop()
```

## Hub Sensors and Feedback

1. **Gyro Turn Using Yaw** – rotate to a heading without counting wheel turns.

```python
hub.motion_sensor.reset_yaw_angle()
desired = 90
while abs(hub.motion_sensor.get_yaw_angle()) < desired:
    drive.start(steering=100, speed=25)
drive.stop()
```

2. **Tilt Reaction** – drive only while the robot is level.

```python
while True:
    if hub.motion_sensor.get_pitch_angle() < 10:  # nearly flat
        drive.start(speed=30)
    else:
        drive.stop()
    hub.wait_for_seconds(0.1)
```

3. **Hub Buttons as Inputs**

```python
hub.light_matrix.write('Go?')
hub.left_button.wait_until_pressed()
drive.move(15, unit='cm', speed=40)
hub.right_button.wait_until_pressed()
drive.stop()
```

4. **Sound Feedback** – play a tone after finishing a task.

```python
from spike import Speaker
speaker = Speaker()
drive.move(20, unit='cm', speed=60)
speaker.beep(60, 0.5)  # middle C for half a second
```

## Mini Challenges

- **Line Patrol** – combine the reflected light loop with distance braking to follow a line until an obstacle appears.
- **Docking Routine** – use a hand wave (Distance Sensor) to start, then drive forward and stop when the Force Sensor feels a bumper press.
- **Color Sorting** – read `color.get_color()` and move a grabber motor to one of three angles based on the color seen.

Experiment by mixing sensors: for example, start with a force press, follow a line with the Color Sensor, keep straight with the gyro, and stop when the Distance Sensor sees a wall. Each snippet here can be chained into longer programs as you get comfortable with the Python API.
