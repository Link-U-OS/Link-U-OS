# Robot Operation Guide

Please read the following content carefully and ensure you understand the controller operation procedures and logic before running on the actual robot. The controller can also be used in simulation environments. It is recommended to familiarize yourself with the controller operation logic in the simulation environment before running on the actual robot.

## Controller Operation Procedures

<font color="red">Before operation, please ensure the robot's emergency stop controller has sufficient battery and can be pressed at any time</font>

1. Press the "start" button on the controller to enter startup mode
	- The terminal in the onboard container should display "start control"
  
![](./images/diagram-1.png)

2. Press the "LB" and "A" buttons simultaneously to enter position control mode
	- The terminal in the onboard container should display "lie2stand"
  
![](./images/diagram-2.png)

<font color="red">At this point, lower the robot, ensure both feet are on the ground, support the robot to maintain balance, and ensure the lifting rope is slack</font>

3. Press the "X" button to enter motion control mode
	- The terminal in the onboard container should display "stand2walk"

![](./images/diagram-3.png)

<font color="red">At this point, ensure the robot's emergency stop controller has sufficient battery and can be pressed at any time, and confirm the environment around the robot is safe</font>

4. Press the "LB" button to enable the control joysticks, both joysticks can send control commands
	- Red arrow corresponds to joystick direction controlling forward and backward movement
	- Green arrow corresponds to joystick direction controlling left and right lateral movement
	- Blue arrow corresponds to joystick direction controlling left and right turning

![](./images/diagram-4.png)

5. Press the "Y" button (circled in green), lift the robot, then press the "start" button (circled in blue) to end robot control

![](./images/diagram-5.png)

## Controller Button Logic
1. "start" button: Switch the robot to enter or exit startup mode
	- Startup mode: Position control for all joints, at this time Kp is very small, the robot should explicitly show that the leg and arm joints receive minimal force, can be moved but with resistance; the terminal in the onboard container should display "start control"
	- Exit startup mode: At this time the robot's leg and arm joints are not under force, terminal displays "stop control"
2. "LB" and "A" buttons: Switch the robot to enter or exit position control mode
	- Enter motion control mode: The robot should explicitly show that the leg and arm joints are under force, can be moved slightly but with resistance; the terminal in the onboard container should display "lie2stand"
	- Exit motion control mode: The robot returns to startup mode, terminal displays "stand2lie"
3. "X" button: Switch the robot to enter motion control mode
	- The robot can stand autonomously; the terminal in the onboard container should display "stand2walk"
4. "Y" button: Switch the robot to exit motion control mode
	- The robot returns to position control mode, terminal displays "walk2stand"
5. "LB" button: Enable control joysticks
	- When this button is pressed, the two joysticks on the controller can send control commands