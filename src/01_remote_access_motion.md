# MentorPi T1 Remote Access and Motion Control

This document describes the first stage of my **Hiwonder MentorPi T1** robot project: connecting to the robot, configuring remote access, launching the ROS2 motion control stack, inspecting ROS2 topics, and manually controlling the tank chassis.

The work was based on the official Hiwonder MentorPi T1 documentation, especially the sections about remote access and motion control:

- Hiwonder MentorPi T1 documentation: https://docs.hiwonder.com/projects/MentorPi-T1/en/latest/
- Remote tool installation and container access: https://docs.hiwonder.com/projects/MentorPi-T1/en/latest/docs/2.remote_tool_installation_and_container_access.html
- Motion control courses: https://docs.hiwonder.com/projects/MentorPi-T1/en/latest/docs/3.motion_control_courses.html

---

## 1. Project Goal

The goal of this stage was to prepare the robot for development from a laptop.

Main tasks:

- connect to the robot remotely
- configure access through the local network
- use VNC for graphical access
- use SSH for terminal access
- start the ROS2 motion control stack
- inspect available ROS2 topics
- move the robot forward
- rotate the robot left and right
- stop the robot safely

This stage is the foundation for later mapping, SLAM, navigation, and autonomous behavior.

---

## 2. Robot Platform

The robot used in this project is the **Hiwonder MentorPi T1**, a ROS2-based tank robot platform.

![Robot ROS2 environment](Screenshots/1_Start/1_Start.png)

During startup, the robot reported the following environment:

```text
Current environment: ROS2
LIDAR: MS200
CAMERA: aurora
MACHINE: MentorPi_Tank
ROS_DOMAIN_ID: 0
VERSION: V1.0.2 | 2026-01-14
```

The hardware and software setup used in this project included:

- Raspberry Pi based onboard computer
- ROS2 environment
- tank chassis
- MS200 LiDAR
- Aurora depth camera
- VNC remote desktop access
- SSH terminal access
- ROS2 motion control stack

The robot is not just a remote-controlled toy. It is a small ROS2 robotics platform where sensors, motor control, robot state, odometry, LiDAR, camera, and user commands are exposed through ROS2 nodes and topics.

---

## 3. How ROS2 Fits Into This Project

ROS2 is the middleware layer that connects different parts of the robot.

Instead of one single program directly controlling everything, ROS2 separates the system into smaller processes called **nodes**.

For example, in this project:

- one node controls the motors
- one node publishes robot state
- one node publishes odometry
- one node publishes IMU data
- one node publishes LiDAR scan data
- another command can publish velocity commands to move the robot

These nodes communicate through **topics**.

A topic is like a named communication channel. One part of the system can publish data, and another part can subscribe to it.

For motion control, the important topic is:

```text
/controller/cmd_vel
```

This topic receives velocity commands. When I publish a velocity command to this topic, the robot controller receives it and moves the tank chassis.

The message type used for movement is:

```text
geometry_msgs/msg/Twist
```

A `Twist` message contains two main parts:

```text
linear  - movement in a straight direction
angular - rotation
```

For this robot:

```text
linear.x > 0   move forward
linear.x < 0   move backward
angular.z > 0  rotate left
angular.z < 0  rotate right
all zeros      stop
```

This is the basic idea behind ROS2 motion control in this project: I do not directly control motors from the terminal. I publish a standard ROS2 velocity message, and the robot controller converts it into motor movement.

---

## 4. Network and Remote Access Setup

The robot can create its own Wi-Fi access point. I first connected to the robot through its default access point and then configured it to connect to my home Wi-Fi network.

After that, both my laptop and the robot were in the same local network.

Final network setup:

```text
Robot LAN IP: 192.168.1.8
SSH user: pi
VNC host: 192.168.1.8
```

SSH connection from the laptop:

```bash
ssh pi@192.168.1.8
```

VNC connection:

```text
192.168.1.8
```

This setup allowed me to work with the robot in two ways:

- **VNC** for graphical tools, desktop, RViz, and visual monitoring
- **SSH** for terminal commands, debugging, and process control

This is useful because robotics development often requires both: a graphical interface for visualization and a terminal for running ROS2 commands.

---

## 5. Starting the ROS2 Motion Control Stack

Before manually launching the motion control stack, I stopped the default robot startup process:

```bash
~/.stop_ros.sh
```

Then I launched the ROS2 controller:

```bash
ros2 launch controller controller.launch.py
```

![Controller launch](Screenshots/1_Start/2_Controller.png)

This started the robot control nodes. The terminal output showed that several ROS2 processes were launched, including:

- robot controller
- robot state publisher
- controller manager
- related hardware/control processes

This step is important because the robot will not respond to `/controller/cmd_vel` commands unless the controller stack is running.

---

## 6. Inspecting ROS2 Topics

After launching the controller, I checked the available ROS2 topics:

```bash
ros2 topic list
```

![ROS2 topic list](Screenshots/1_Start/3_Ros2_topic_list.png)

Important topics included:

```text
/app/cmd_vel
/cmd_vel
/controller/cmd_vel
/controller_manager/joint_states
/diagnostics
/imu
/imu/data_raw
/joint_states
/odom
/odom_raw
/parameter_events
/robot_description
/ros_robot_controller/battery
/ros_robot_controller/bus_servo/set_position
/ros_robot_controller/bus_servo/set_state
/ros_robot_controller/button
/ros_robot_controller/enable_reception
/ros_robot_controller/imu_raw
/ros_robot_controller/joy
/ros_robot_controller/set_buzzer
/ros_robot_controller/set_led
/ros_robot_controller/set_motor
/ros_robot_controller/set_oled
/ros_robot_controller/set_rgb
/scan_raw
/tf
/tf_static
```

The most important topic for this stage was:

```text
/controller/cmd_vel
```

This is the topic used to send movement commands to the robot.

---

## 7. Manual Movement Control

The tank chassis was controlled by publishing `geometry_msgs/msg/Twist` messages to:

```text
/controller/cmd_vel
```

### 7.1 Move Forward

To move the robot forward slowly, I used:

```bash
ros2 topic pub /controller/cmd_vel geometry_msgs/msg/Twist \
"{linear: {x: 0.1, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}" --once
```

![Move forward command](Screenshots/1_Start/4_Move_forward.png)

Video demonstration:

<video controls src="Video/1_Movement.MOV" title="MentorPi T1 forward movement"></video>

If the embedded video does not render in GitHub, use this link:

[Forward movement video](Video/1_Movement.MOV)

The value:

```text
linear.x = 0.1
```

means slow forward movement.

I used a low speed because this was the first motion test, and it is safer to start slowly when testing a robot on the floor.

---

### 7.2 Stop the Robot

To stop the robot, I sent a zero velocity command:

```bash
ros2 topic pub /controller/cmd_vel geometry_msgs/msg/Twist \
"{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}" --once
```

This command is important.

Stopping a terminal process with `Ctrl+C` does not always guarantee that the robot receives a final stop command. The safer approach is to explicitly publish a `Twist` message with all velocity values set to zero.

---

### 7.3 Rotate Left

To rotate the robot left, I used:

```bash
ros2 topic pub /controller/cmd_vel geometry_msgs/msg/Twist \
"{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.5}}" --once
```

Here:

```text
angular.z = 0.5
```

means slow left rotation.

---

### 7.4 Rotate Right

To rotate the robot right, I used:

```bash
ros2 topic pub /controller/cmd_vel geometry_msgs/msg/Twist \
"{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: -0.5}}" --once
```

Here:

```text
angular.z = -0.5
```

means slow right rotation.

![Rotate command](Screenshots/1_Start/5_Move_rotate.png)

Video demonstration:

<video controls src="Video/2_Rotate.MOV" title="MentorPi T1 rotation test"></video>

If the embedded video does not render in GitHub, use this link:

[Rotation video](Video/2_Rotate.MOV)

---

## 8. Why `/controller/cmd_vel` Is Used

There may be several velocity-related topics in a ROS2 robot system, for example:

```text
/cmd_vel
/app/cmd_vel
/controller/cmd_vel
```

In this setup, the robot responded to:

```text
/controller/cmd_vel
```

This means that the active low-level controller was listening to that topic for movement commands.

The general control chain looked like this:

```text
Terminal command
      ↓
ros2 topic pub
      ↓
/controller/cmd_vel
      ↓
geometry_msgs/msg/Twist
      ↓
robot controller node
      ↓
motor control
      ↓
tank chassis movement
```

This is a useful ROS2 concept: the user does not directly control the motors. Instead, the user publishes a standard message to a topic, and the robot controller handles the hardware-level execution.

---

## 9. Safety Notes

During testing, I used low speeds first:

```text
linear.x = 0.1
angular.z = 0.5
```

Safety practices used during the test:

- keep the robot on the floor
- start with very low speed
- keep a stop command ready
- avoid running multiple conflicting control processes
- stop the robot before closing terminals
- avoid testing near stairs, cables, or fragile objects
- monitor battery level
- stop testing if the robot starts beeping

The robot later started beeping during a heavier SLAM session, most likely because of low battery or power load. I stopped the test and placed the robot on charge.

---

## 10. What I Learned

In this stage, I learned how to:

- connect to a ROS2 robot through SSH and VNC
- configure the robot for local network development
- use VNC for graphical access
- use SSH for command-line access
- stop the default robot service before manual testing
- launch the ROS2 motion controller
- inspect ROS2 topics
- identify the correct velocity command topic
- publish `geometry_msgs/msg/Twist` messages
- control forward movement
- control left and right rotation
- stop the robot safely
- understand the basic ROS2 topic-based control model

The most important practical result was that I successfully moved the robot using ROS2 commands instead of a mobile app or joystick.

---

## 11. Current Status

Completed:

```text
Remote access through LAN: done
SSH access: done
VNC access: done
ROS2 controller launch: done
ROS2 topic inspection: done
Manual forward movement: done
Manual rotation: done
Safe stop command: done
Video evidence: recorded
```

Next stage:

```text
LiDAR-based SLAM mapping
RViz visualization
Map saving
Map inspection
Autonomous navigation using the saved map
```