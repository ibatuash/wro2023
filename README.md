# WRO notes
This is an assorted collection of things that we learned by doing a WRO challenge for the first time. 

## Basic movement
### Turns
There are 2 parameters in the turn function: 'direction' (from -100 to +100) and 'distance'. Distance means how much should the 'main' motor move. 'Main' motor is Left for negative numbers and Right for positive numbers. 
Distance can be in: 
- degrees (useful for small adjustments)
- rotations (useful for large movements or when the motor is operating some mechanism)
- cm/inch (converted into degrees based on the wheel size. If non-standard wheels are used, need to set at the start of the program the wheel diameter). 

Direction means how much of the total is assigned to each wheel. 
For example:
- Direction: +100%, distance: 360 degrees. Only the right motor turns clockwise for 360 degrees, left motor is not moved
- Direction: +50%, distance: 360 degrees. Right motor turns clockwise 360 degrees, the left motor turns 180% degrees. 
This number changes the central point around which the robot rotates. 

https://www.youtube.com/watch?v=fAr3o0mMvY4

### Follow the line (until condition)
The idea is to pick a fixed reflection number (for example 50%) and adjust movement to keep it the same. This number needs to be between black and white reflection numbers - so important to keep lighting the same. Also the reflection number adjusts slightly the target position of the sensor. 
Movement is done by changing the speed with which the wheels turn, adding corrections based on lighting. 
Amount correction changes how fast the number is adjusted. 
Too large number: large correction angles
Too small correction: long 'waves' of correction (unpredictable end position) 

## Advanced movement:
### Follow the line (distance) 
Sometimes it is useful to follow the line for a given distance. This can be done by making the 'stop' condition based on the motor position. The idea is: before moving, reset wheel 'position' to 0. If you need to move by 10 cm, you can calculate how much this is in degrees (or rotations), and use the motor "current angle" position as a stopping criteria.
Video reference:

### Alignment on the line (reflection difference) 
If there are 2 reflection sensors in the build, they can be used to align the robot on a line, when coming to it from some angle. Idea is to make the reflection number the same (within some tolerance).
 
### Compass (AKA "yaw" giro parameter) 
Robot hub also has a built-in direction sensor. It is confusingly called "yaw" - but you can think of it as a compass. The parameter value ranges from -180 to +180.  It needs to be set to zero when the program starts. Then after turns, you can use the current compass value to adjust rotation: if the compass value is 88 - make an extra 2 degree turn (this logic can be integrated into a custom function such as "turn left" or "turn right". 
Compass will accumulate error after multiple turns, so it is useful to 'reset' it during the program if robot position is known: for example, after aligning on the line - or intentionally bumping into the wall to adjust robot position) 
https://www.youtube.com/watch?v=YMk6lN55eBQ
https://www.youtube.com/watch?v=uL9V4OcNs5U

## Hardware general design:
- Light sensors: black-to-white range depends on lighting conditions. In theory, light sensors output numbers from 0% (black) to 100% (white). In practice, this depends on background lighting conditions. With good lighting, black color can be 30% and white is 100%. With dim lighting, black can be 20% and white 50%. This range matters a lot for "follow the line" logic. 
- Robot stability: it is better keep the center of gravity low. This adds stability, and helps to reduce random errors on turns or movements
- Wheel traction: Wheels from the Spike set can slip with high acceleration. WRO teams generally use larger / wider wheels (from other Lego sets) to have more traction. Even with the same surface, traction can depend on the surface below the playing field. When were testing on the map with a soft carpet underneath it, there is more traction versus the same map, placed on the hard surface (wooden table). On the soft underlying surface, the robot pushes down the map with its own map and increases contact area for the wheels.
- New "Spike" hubs have 6 sockets only. Out of these: 
-- 2 are needed for wheels
-- 1 or 2 for light sensors (at least one is needed for tracking, and second one is very likely needed for crossroad detection). 
-- Then, distance sensor is likely to be needed to detect objects on the map. 
-- After all this, this leaves only 1 socket for the arm motor. 
This means that older EV3 hubs have a clear advantage - as they can use one more additional motor for the secondary arm (or additional sensor).

## Hardware hints: 
- Distance sensor:
The sensor reading is relatively slow (compared to light sensors), so it is useful to reduce speed if moving until some distance is expected. 
- Generally, all motors and sensors can have slight tolerance / differences. So it is useful to label each sensor motor and be sure they are installed in the same place.
- Robot start position should be same. This means that errors are similar across the runs. If using compass / yaw logic, make sure the robot alignment is exact 
- Set movable arm position at the start of the program
- Position of the sensor matters (upside down for distance), or rotation for light sensors 

## Program organization:
### Basic movement/action functions 
You would likely need to have functions for turns, line following, basic arm actions. Even if an action requires a single Spike block (for example for turning), it is still useful to make it a custom function: you may want to adjust the numbers later, or add some logic (like changing speed or adding corrections) 

### Functions need conventions/expectations for start / end positions 
Functions generally rely on robot being in a certain position before being called. For example, 'following line' can assume that robot is indeed on the line already. Or, when executing a function for TaskA, robot ends up in some position. This position then assumed in calling next function for TaskB

### Layout of the functions
Functions should be visually organised 
- Names of the functions with prefixes (sorted) 
- Reusable vs easy-to-tweak functions 
- Avoid copy-paste 

## Finding issues (debugging):
- Spike 
- insert delays
- screen output (writing text/variables adds pauses!)
- button color
- wait for button press
- modular test programs 

## Spike/HUB issues / bugs:
- Always check battery / connections: with low charge ( < 30%), hub can misbehave (it looks sensor readings can be slower, some commands get stuck/delayed)
- If Spike App is slow, it is sometimes help to restart spike app (and/or PC itself)
- We have noticed that using a Bluetooth mouse causes a lot of slow-down and issues when running programs. So better to avoid using Bluetooth for anything else at the same time.
- Copying functions from other projects is possible but looks to be buggy (at least in Spike v3.2.4). Be careful if copied functions use any variables (likely they will need to be re-created with a new name in the target project). Also existing variables can stop working (!). Also check, if the copied functions call other (non-copied) functions - in this case Spike silently fails to start the program - "Play" button does not do anything. (only printing errors in the log file).
- There is a Spike App log, located at XXXXXXXXXX log file location (Windows). This can reveal further issues.
