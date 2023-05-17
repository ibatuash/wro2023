# WRO notes
This is an assorted collection of things that we learned by doing a WRO challenge (2023, Elementary) for the first time. 

# Hardware
## Robot design
- Try to make the design as simple as possible - and practice disassembling / reassembling the robot several times before competition. Small changes or mistakes in the design can break expectations in the program (for example, if position of distance sensor changes).
- Robot stability: it is better keep the center of gravity low. This adds stability, and helps to reduce random errors on turns or movements
- Wheel traction: Wheels from the Spike set can slip with high acceleration. WRO teams generally use larger / wider wheels (from other Lego sets) to have more traction. Even with the same surface, traction can depend on the surface below the playing field. When we were testing on the playin field with a soft carpet underneath it, there is more traction versus the same surface, placed on the hard underlying (wooden table). The reason for this, is that on the soft underlying surface, the robot pushes down the map with its own weight - this increases contact area for the wheels and therefor traction.

## Sensors and motors
- Light sensors: In theory, light sensors output numbers from 0% (black) to 100% (white). In practice, this depends on the background lighting conditions: with good lighting, black color can be 30% and white is 100%. With dim lighting, black can be 20% and white 60%. This range matters a lot for "follow the line" logic. 
- New "Spike" hubs have 6 connection sockets only. Out of these: 
-- 2 are needed for wheels
-- 1 or 2 are needed for light sensors (certainly 1 is needed for tracking, and second one is very likely needed for crossroad detection). 
-- 1 distance sensor is likely to be needed to detect objects on the playing field
-- After all this, this leaves only 1 socket for the arm motor. It means that there is generally one type of movement which arm can do.
So older EV3 hubs with 8 sockets have a clear advantage - as they can use more  motors for the secondary arm (or additional sensor).

## Hardware hints
- Distance sensor: The sensor reading is relatively slow (compared to light sensors), so if it is read while moving (for example, moving until detecting some object), it is useful to reduce speed to have less error.
- Generally, all motors and sensors can have slight tolerance / differences. So it is useful to label each sensor motor and be sure they are installed in the same place. For example, it is easy to put a distance sensor upside down - this will generally work, but can introduce small differences. 
- Robot start position should be same: this means that errors are similar across the runs. If using compass / yaw logic, make sure the robot alignment is exact.
- Set movable arm position at the start of the program (different arm position can change center of mass - or some function can assume it is in some position)

# Programming (Spike)
## Basic movement
### Standard "turn" block
There are 2 parameters in the Spike turn function: 'direction' (from -100 to +100) and 'distance'. Distance means how much should the 'Main' motor move. 'Main' motor is Left for negative numbers and Right for positive numbers. 
'Distance' can be set in: 
- Degrees (useful for small adjustments)
- Rotations (useful for large movements or when the motor is operating some mechanism)
- Cm/Inch (converted into degrees, based on the wheel size. If non-standard wheels are used, it is needed to set the wheel diameter at the start of the program). 

'Direction' means how much of the total is assigned to each wheel. 
For example:
- Direction: +100%, distance: 360 degrees: Only the right motor turns clockwise for 360 degrees, left motor is not moved
- Direction: +50%, distance: 360 degrees: Right motor turns clockwise 360 degrees, the left motor turns 180% degrees. 
This number changes the central point around which the robot rotates. 

https://www.youtube.com/watch?v=fAr3o0mMvY4

### Following lines (until some condition)
Playing fields have lines (usially in black-white-black pattern) which help robot to navigate. The idea of following the line is is to pick a target reflection number (for example 50%) and adjust movement to keep it the same. This number needs to be between black and white reflection numbers - so important to keep lighting the same. Also the reflection number adjusts slightly the target position of the sensor. Then movement is done by adjusting the speed with which the wheels turn, adding corrections based on sensor reading. For this, a Spike command from "Extensions -> More Movement" section is needed (you need to enable it first at the bottom left corner). Here is an example of a function which follows black line until distance senrs sees an obstacle within "cm" distance.

![image](https://github.com/ibatuash/wro2023/assets/134078/a95d1319-7aa6-4b3f-803c-05891a23425f)


Amount correction changes how fast the number is adjusted. 
Too large number: large correction angles
Too small correction: long 'waves' of correction (unpredictable end position) 

Links: https://ev3lessons.com/en/ProgrammingLessons/advanced/LineFollower.pdf

## Advanced movement
### Following lines (for a predefined distance) 
Sometimes it is useful to follow the line for a given distance. This can be done by making the 'stop' condition based on the motor position. The idea is: before moving, reset wheel 'position' to 0. If you need to move by 10 cm, you can calculate how much this is in degrees (or rotations), and use the motor "current angle" position as a stopping criteria.
Video reference: TODO

### Alignment on the line (reflection difference) 
If there are 2 reflection sensors in the build, they can be used to align the robot on a line, when coming to it from some angle. The idea is to make the reflection number the same (within some tolerance). The exact correction logic depends on the robot design (ie whether sensors are parallel/perpendicular to the wheels, and what is the distance to the wheels)
 
### Compass (AKA "yaw" gyroscope parameter) 
Robot hub also has a built-in direction sensor. It is confusingly called "yaw" - but you can think of it as a compass. The parameter value ranges from -180 to +180.  Zero (reference) position needs to be set when the program starts. Then after make turns, you can use the current compass value to adjust rotation: if the compass value is 88 - make an extra 2 degree turn (this logic can be integrated into a custom function such as "turn left" or "turn right". Compass will accumulate error after multiple turns, so it is useful to 'reset' it during the program when robot position is known: for example, after aligning on the line - or intentionally bumping into the wall) 
https://www.youtube.com/watch?v=YMk6lN55eBQ
https://www.youtube.com/watch?v=uL9V4OcNs5U

## Program organization
### Basic movement/action functions 
You would likely need to have basic reusable functions for turns, line following, basic arm actions. Even if an action requires a single Spike block (for example for turning, or raising the arm), it is still useful to make it a custom function: you may want to adjust it later (if you change physical arm design), for example to add some logic (like changing speed or adding corrections).

### Functions need conventions/expectations for start / end positions 
Functions generally rely on the robot being in a certain position before they are called. For example, 'following line' can assume that robot is indeed on the line already. Or, when executing a function for some TaskA, robot ends up in some PositionA. Then the next function for TaskB would need to start from PositionA.

### Layout of the functions
- It is helpful to visually organize related functions together. There can be an area for "basic" functions, then left-to-right organized functions (in the approximate order in which they are called).
- Spike sorts functions by name. So it is useful to have same prefixes for related functions, for example: "move_until_crossroad", "move_by_distance", "taskA_subtaskX", "taskA_subtaskY", etc
- Reusable vs easy-to-tweak functions: Functions should be composable, and composition of the functions should be easy to adjust. For example, there are 'surprise tasks' announced on the day of the competition: you should be able to change a particular step in some task without rearranging / changing multiple places
- Avoid copy-pasting functions: if you need to adjust some logic, it is easier to do in one place. If the same value is used in multiple places (for example, speed or arm position), can use a variable to refer to it.

## Finding issues (debugging)
Basic steps to fix errors: **Reproduce** (more than once!), **Understand**, **Fix**, **Test** (more than once!).
Finding issues can be difficult, especially if they are not always reproducible. It is very important to be effective in investigation of the issues - there will be many!

- Most basic way: Reduce robot speed / insert Wait (before problematic step) or "Stop all" (after problematic step) blocks. If robot gets lost after some specific place, for example, a specific turn: add "sleep" block to see how exactly it stops (what are the sensor readings), in which part of the program, and what are the next blocks. You can use "wait for button press" logic to resume program after you analyzed the error
- Use screen output to display what robot is doing. Drawing patterns or pixels is fast, however outputting text/variables can add small pauses in the program - this can also change the behavior (for example if done when moving). 
- Try to use as small as possible test program to reproduce errors. If a mistake happens after 1 minute of the running program, making 10 attempts to fix costs extra 10 minutes.
- After fixing problem in isolation, test it with the main program

## Spike/HUB issues / bugs
- Always check battery / connections: with low charge ( < 30%), hub can misbehave (it looks sensor readings can be slower, some commands get stuck/delayed)
- If Spike App is slow, it is sometimes help to restart spike app (and/or PC itself)
- We have noticed that using a Bluetooth mouse causes a lot of slow-down and issues when running programs. So better to avoid using Bluetooth for anything else at the same time.
- Copying functions from other projects is possible is very buggy (at least in Spike v3.2.4). Be careful to the points: 
-- If copied functions use any variables (likely they will need to be re-created with a new name in the target project). Also existing variables can stop working (!). 
-- Program can be "pasted" into a random (?) place - and if you press Ctrl-V 2 times it make a duplcate, and the project gets broken
-- If the copied functions call other (non-copied) functions - in this case Spike silently fails to start the program - "Play" button does not do anything. (only printing errors in the log file).
- There is a Spike App log, located at XXXXXXXXXX log file location (Windows). This can reveal further issues.
