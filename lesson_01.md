# Lesson 01: Motors and Color Sensor Basics

This lesson introduces the fundamental concepts of controlling the LEGO Spike Prime motors and using the color sensor. The focus is on practicing block-based programming while also showing how the same ideas translate into Python. Start with the simple examples below to see the building blocks before moving on to the block-based exercises.

## Python Warmups

All examples assume:

```python
from spike import PrimeHub, Motor, MotorPair, ColorSensor

hub = PrimeHub()
motors = MotorPair('C', 'D')
sensor = ColorSensor('E')
```

1. **Move Forward for One Second**

```python
motors.set_default_speed(50)
motors.start()
hub.wait_for_seconds(1)
motors.stop()
```

2. **Turn in Place (Left)**

```python
motors.start(steering=-100, speed=40)  # negative steering spins left
hub.wait_for_seconds(1)
motors.stop()
```

3. **Stop When Red Appears**

```python
motors.start(speed=40)
while sensor.get_color() != 'red':
    hub.wait_for_seconds(0.1)
motors.stop()
```

4. **Gradual Speed Change**

```python
for speed in range(20, 101, 20):
    motors.start(speed=speed)
    hub.wait_for_seconds(0.5)
motors.stop()
```

5. **Hub Light Responds to Colors**

```python
while True:
    seen = sensor.get_color()
    if seen:
        hub.light_matrix.set_orientation(0)  # keeps lights upright on the desk
        hub.light_matrix.show_image('ARROW_N')
        hub.status_light.on(seen)
    hub.wait_for_seconds(0.2)
```

Use these warmups to get comfortable with the Python API; the block exercises below mirror the same behaviors.

## Exercises

1. **Move Forward** - Program the robot to drive forward for one second.

```text
when program starts
  set driving motors to C+D
  set drive speed to 50%
  start moving forward
  wait 1 second
  stop driving
```

2. **Move Backward** - Make the robot drive backward for one second.

```text
when program starts
  set driving motors to C+D
  set drive speed to 50%
  start moving backward
  wait 1 second
  stop driving
```

3. **Turn Left** - Create a program that turns the robot 90 degrees to the left.

```text
when program starts
  turn left 90 degrees
  stop driving
```

4. **Turn Right** - Create a program that turns the robot 90 degrees to the right.

```text
when program starts
  turn right 90 degrees
  stop driving
```

5. **Stop at Color** - Drive forward and stop when the color sensor detects red.

```text
when program starts
  start moving forward
  wait until color sensor sees red
  stop driving
```

6. **Light Up** - Use the color sensor's built-in light to change colors while moving.

```text
when program starts
  repeat while moving
    cycle sensor light colors
  end
```

7. **Reverse at Edge** - Drive forward until the color sensor detects black and then reverse.

```text
when program starts
  start moving forward
  wait until color sensor sees black
  start moving backward
  stop after 1 second
```

8. **Spin in Place** - Program both motors to spin in opposite directions for a quick spin.

```text
when program starts
  set left motor forward
  set right motor backward
  wait 1 second
  stop driving
```

9. **Variable Speed** - Change the motor speed dynamically while driving forward.

```text
when program starts
  set speed to 20%
  start moving forward
  change speed gradually to 100%
  stop driving
```

10. **Short Delay** - Insert a one-second wait between two movements.

```text
when program starts
  move forward for 1 second
  wait 1 second
  move backward for 1 second
```

11. **Continuous Loop** - Run a loop to keep the robot driving forward until a button is pressed.

```text
when program starts
  while button not pressed
    drive forward
  end
  stop driving
```

12. **Color Response** - Display a different light color on the hub for each color the sensor reads.

```text
when color sensor changes
  if red then set hub light to red
  else if blue then set hub light to blue
  else set hub light to green
```

13. **Precise Distance** - Drive forward for a set number of motor rotations.

```text
when program starts
  reset motor rotation
  start moving forward
  wait until rotation > target
  stop driving
```

14. **Gradual Stop** - Slow down the motors gradually before coming to a stop.

```text
when program starts
  start moving forward
  decrease speed over 2 seconds
  stop driving
```

15. **Dance Routine** - Combine several movements and color changes to create a simple dance.

```text
when program starts
  move forward
  change light to blue
  turn right
  change light to yellow
  move backward
  spin in place
  stop driving
```

