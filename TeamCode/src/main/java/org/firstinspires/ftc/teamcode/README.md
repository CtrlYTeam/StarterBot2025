# GoBilda

Official goBILDA learning code from [GitHub](https://github.com/goBILDA-Official/FtcRobotController-Add-Starter-Kit-Code/tree/Add-Starter-Kit-Code).

## TeamCode Module

This module, TeamCode, is the place where you will write/paste the code for your team's
robot controller App.

## Creating your own OpModes

The easiest way to create your own OpMode is to copy a Sample OpMode and make it your own. 
Sample opmodes exist in the [FtcRobotController module](https://github.com/goBILDA-Official/FtcRobotController-Add-Starter-Kit-Code/tree/Add-Starter-Kit-Code/FtcRobotController/src/main/java/org/firstinspires/ftc/robotcontroller/external/samples).

### Starter Kit code

This OpMode is an example driver-controlled (TeleOp) mode for the goBILDA 2024-2025 FTC **Into The Deep Starter Robot**

The code is structured as a LinearOpMode:

This robot has a two-motor differential-steered (sometimes called tank or skid steer) drivetrain.
With a left and right drive motor.
The drive on this robot is controlled in an "Arcade" style, with the left stick Y axis controlling the forward movement and the right stick X axis controlling rotation.
This allows easy transition to a standard "First Person" control of a mecanum or omnidirectional chassis.

The drive wheels are 96mm diameter traction (Rhino) or omni wheels. They are driven by 2x 5203-2402-0019 312RPM Yellow Jacket Planetary Gearmotors.

This robot's main scoring mechanism includes an arm powered by a motor, a "wrist" driven by a servo, and an intake driven by a continuous rotation servo.

The arm is powered by a 5203-2402-0051 (50.9:1 Yellow Jacket Planetary Gearmotor) with an external 5:1 reduction. This creates a total ~254.47:1 reduction.
This OpMode uses the motor's encoder and the RunToPosition method to drive the arm to specific setpoints. These are defined as a number of degrees of rotation away from the arm's 
starting position.

Make super sure that the arm is reset into the robot, and the wrist is folded in before you run start the OpMode. The motor's encoder is "relative" and will move the number of degrees 
you request it to based on the starting position. So if it starts too high, all the motor setpoints will be wrong.

The wrist is powered by a goBILDA Torque Servo (2000-0025-0002).

The intake wheels are powered by a goBILDA Speed Servo (2000-0025-0003) in Continuous Rotation mode.


ARM_TICKS_PER_DEGREE constant is the number of encoder ticks for each degree of rotation of the arm.
To find this, we first need to consider the total gear reduction powering our arm.
First, we have an external 20t:100t (5:1) reduction created by two spur gears.
But we also have an internal gear reduction in our motor.
The motor we use for this arm is a 117RPM Yellow Jacket. Which has an internal gear reduction of ~50.9:1. (more precisely it is 250047/4913:1)
We can multiply these two ratios together to get our final reduction of ~254.47:1.
The motor's encoder counts 28 times per rotation. So in total you should see about 7125.16 counts per rotation of the arm. We divide that by 360 to get the counts per degree.

ARM_COLLAPSED_INTO_ROBOT, ARM_COLLECT, ARM_CLEAR_BARRIER, ARM_SCORE_SPECIMEN, ARM_SCORE_SAMPLE_IN_LOW, ARM_ATTACH_HANGING_HOOK, ARM_WINCH_ROBOT constants hold the position that 
the arm is commanded to run to. These are relative to where the arm was located when you start the OpMode. So make sure the arm is reset to collapsed inside the robot before you start the program.

ARM_COLLAPSED_INTO_ROBOT, ARM_CLEAR_BARRIER, ARM_SCORE_SPECIMEN, ARM_SCORE_SAMPLE_IN_LOW, ARM_ATTACH_HANGING_HOOK, ARM_WINCH_ROBOT In these variables you'll see a number in degrees, multiplied by the ticks per degree of the arm. This results in the number of encoder ticks the arm needs to move in order to achieve the ideal
set position of the arm. For example, the ARM_SCORE_SAMPLE_IN_LOW is set to 160 * ARM_TICKS_PER_DEGREE. This asks the arm to move 160° from the starting position.
If you'd like it to move further, increase that number. If you'd like it to not move as far from the starting position, decrease it.

INTAKE_COLLECT, INTAKE_OFF, INTAKE_DEPOSIT Variables to store the speed the intake servo should be set at to intake, and deposit game elements.

WRIST_FOLDED_IN, WRIST_FOLDED_OUT Variables to store the positions that the wrist should be set to when folding in, or folding out.

FUDGE_FACTOR A number in degrees that the triggers can adjust the arm position by.

ARM_COLLAPSED_INTO_ROBOT Variables that are used to set the arm to a specific position.

Most skid-steer/differential drive robots require reversing one motor to drive forward. For this robot, we reverse the right motor.

Setting zeroPowerBehavior to BRAKE enables a "brake mode". This causes the motor to slow down much faster when it is coasting. 
This creates a much more controllable drivetrain. As the robot stops much quicker.

Before starting the armMotor. We'll make sure the TargetPosition is set to 0. Then we'll set the RunMode to RUN_TO_POSITION. And we'll ask it to stop and reset encoder.
If you do not have the encoder plugged into this motor, it will not run in this code.

Set the drive and turn variables to follow the joysticks on the gamepad. The joysticks decrease as you push them up. So reverse the Y axis.

Here we "mix" the input channels together to find the power to apply to each motor. The both motors need to be set to a mix of how much you're retesting the robot move
forward, and how much you're requesting the robot turn. When you ask the robot to rotate the right and left motors need to move in opposite directions. So we will add rotate to
forward for the left motor, and subtract rotate from forward for the right motor.

Here we handle the three buttons that have direct control of the intake speed.
These control the continuous rotation servo that pulls elements into the robot,
If the user presses A, it sets the intake power to the final variable that
holds the speed we want to collect at.
If the user presses X, it sets the servo to Off.
And if the user presses B it reveres the servo to spit out the element.*/

TECH TIP: If Else statements:
We're using an else if statement on "gamepad1.x" and "gamepad1.b" just in case multiple buttons are pressed at the same time. 
If the driver presses both "a" and "x" at the same time. "a" will win over and the intake will turn on. If we just had
three if statements, then it will set the intake servo's power to multiple speeds in one cycle. Which can cause strange behavior. */

Here we implement a set of if else statements to set our arm to different scoring positions.
We check to see if a specific button is pressed, and then move the arm (and sometimes
intake and wrist) to match. For example, if we click the right bumper we want the robot
to start collecting. So it moves the armPosition to the ARM_COLLECT position,
it folds out the wrist to make sure it is in the correct orientation to intake, and it
turns the intake on to the COLLECT mode.

Here we create a "fudge factor" for the arm position.
This allows you to adjust (or "fudge") the arm position slightly with the gamepad triggers.
We want the left trigger to move the arm up, and right trigger to move the arm down.
So we add the right trigger's variable to the inverse of the left trigger. If you pull
both triggers an equal amount, they cancel and leave the arm at zero. But if one is larger
than the other, it "wins out". This variable is then multiplied by our FUDGE_FACTOR.
The FUDGE_FACTOR is the number of degrees that we can adjust the arm by with this function.

Here we set the target position of our arm to match the variable that was selectedby the driver.
We also set the target velocity (speed) the motor runs at, and use setMode to run it.

TECH TIP: Encoders, integers, and doubles
Encoders report when the motor has moved a specified angle. They send out pulses which
only occur at specific intervals (see our ARM_TICKS_PER_DEGREE). This means that the
position our arm is currently at can be expressed as a whole number of encoder "ticks".
The encoder will never report a partial number of ticks. So we can store the position in an integer (or int).
A lot of the variables we use in FTC are doubles. These can capture fractions of whole
numbers. Which is great when we want our arm to move to 122.5°, or we want to set our servo power to 0.5.

setTargetPosition is expecting a number of encoder ticks to drive to. Since encoder ticks are always whole numbers, it expects an int. 
But we want to think about our arm position in degrees. And we'd like to be able to set it to fractions of a degree.
So we make our arm positions Doubles. This allows us to precisely multiply together armPosition and our armPositionFudgeFactor. 
But once we're done multiplying these variables. We can decide which exact encoder tick we want our motor to go to. We do this 
by "typecasting" our double, into an int. This takes our fractional double and rounds it to the nearest whole number.

