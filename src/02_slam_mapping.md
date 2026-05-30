# MentorPi T1 LiDAR SLAM Mapping

This document describes the second stage of my **Hiwonder MentorPi T1** robot project: building a 2D map of a room using LiDAR, ROS2 SLAM, RViz visualization, and keyboard teleoperation.

In the previous stage, I configured remote access to the robot, connected to it through SSH and VNC, launched the ROS2 motion controller, and tested manual movement commands. In this stage, I used the robot’s LiDAR to generate and save an occupancy grid map.

The work was based on the official Hiwonder MentorPi T1 documentation:

- Hiwonder MentorPi T1 documentation: https://docs.hiwonder.com/projects/MentorPi-T1/en/latest/
- Mapping courses: https://docs.hiwonder.com/projects/MentorPi-T1/en/latest/docs/6.mapping_courses.html
- Navigation lesson: https://docs.hiwonder.com/projects/MentorPi-T1/en/latest/docs/7.navigation_lesson.html

---

## 1. Goal

The goal of this stage was to test the full LiDAR SLAM workflow:

1. Launch the ROS2 SLAM stack.
2. Visualize the robot and LiDAR data in RViz.
3. Drive the robot around the room using keyboard teleoperation.
4. Generate a 2D occupancy grid map.
5. Save the map as `.pgm` and `.yaml` files.
6. Prepare the saved map for future navigation testing.

The final selected map for this stage is:

```text
maps/mentorpi_map_02.pgm
maps/mentorpi_map_02.yaml
```

I also kept the first map metadata file:

```text
maps/mentorpi_map_01.yaml
```

The first mapping run produced a larger but noisy map. I kept it as an intermediate result and comparison point, but the second one-room map was selected as the final result for this project stage.

---

## 2. Robot and Environment

The robot used in this project is the **Hiwonder MentorPi T1**, running a ROS2 environment.

During startup, the robot reported the following configuration:

```text
Current environment: ROS2
LIDAR: MS200
CAMERA: aurora
MACHINE: MentorPi_Tank
ROS_DOMAIN_ID: 0
VERSION: V1.0.2 | 2026-01-14
```

The relevant hardware and software for this stage:

- MentorPi T1 tank chassis
- Raspberry Pi onboard computer
- ROS2 environment
- MS200 LiDAR
- Aurora depth camera
- VNC remote desktop access
- SSH terminal access
- RViz visualization

The robot was connected to my home Wi-Fi network and accessed remotely:

```text
Robot LAN IP: 192.168.1.8
SSH user: pi
VNC host: 192.168.1.8
```

---

## 3. How LiDAR SLAM Works

SLAM means **Simultaneous Localization and Mapping**.

The robot tries to do two things at the same time:

1. Estimate where it is.
2. Build a map of the environment around it.

For this project, the robot used its LiDAR sensor to scan the room. The LiDAR produces distance measurements around the robot. These measurements are published as ROS2 scan data.

The important LiDAR topic was:

```text
/scan_raw
```

The SLAM system uses this scan data together with robot movement information and coordinate transforms to generate a map.

The generated map is published as:

```text
/map
```

The output is an **occupancy grid map**. In this map:

```text
white / light gray  - free space
black               - obstacle or wall
gray                - unknown area
```

This is the standard type of 2D map used by many ROS navigation systems.

---

## 4. ROS2 Components Used

The mapping pipeline can be summarized like this:

```text
MS200 LiDAR
    ↓
/scan_raw
    ↓
SLAM node
    ↓
/map
    ↓
RViz visualization
```

Other important ROS2 topics and concepts:

```text
/odom       - robot odometry
/tf         - coordinate transforms
/tf_static  - static coordinate transforms
```

RViz does not create the map by itself. It only visualizes the data produced by the running ROS2 nodes.

---

## 5. Starting the Mapping Session

Before launching SLAM manually, I stopped the default robot process:

```bash
~/.stop_ros.sh
```

Then I launched the SLAM node:

```bash
ros2 launch slam slam.launch.py
```

This terminal stayed open during the whole mapping session.

---

## 6. Launching RViz

In a second terminal, I launched RViz with the SLAM configuration:

```bash
ros2 launch slam rviz_slam.launch.py
```

![SLAM start in RViz](Screenshots/2_Mapping/1_Start.png)

In RViz, I could see:

- robot model
- LiDAR scan points
- generated occupancy grid
- robot pose
- map updates
- transform visualization

The colored points represent live LiDAR readings. The black and white grid represents the generated map.

---

## 7. Keyboard Teleoperation

To drive the robot while mapping, I used keyboard teleoperation:

```bash
ros2 launch peripherals teleop_key_control.launch.py
```

The teleoperation window showed:

```text
W - move forward
S - move backward
A - turn left
D - turn right
Ctrl+C - quit
```

![Starting movement during mapping](Screenshots/2_Mapping/2_Start_moving.png)

This method was more convenient than sending individual velocity commands because I could move the robot while watching the map update in RViz.

---

## 8. Driving Strategy

For the final mapping attempt, I used a smaller area: one room instead of a larger route.

This was done because the first mapping attempt produced a larger but noisy map. A smaller room is easier to map because there are fewer localization errors and fewer chances for the tank tracks to slip.

The driving strategy was:

```text
1. Start from a stable position.
2. Wait a few seconds before moving.
3. Move forward slowly.
4. Stop often.
5. Rotate slowly.
6. Avoid fast turns.
7. Avoid track slipping.
8. Do not lift or manually move the robot during mapping.
9. Keep the route small and controlled.
```

I also recorded a screen video while driving the robot for mapping:

<video controls src="Video/3_Mapping.mov" title="MentorPi T1 LiDAR SLAM mapping"></video>


---

## 9. Mapping in Progress

During the mapping run, RViz displayed the live occupancy grid and LiDAR scan points.

![SLAM working in RViz](Screenshots/2_Mapping/3_Working.png)

In the visualization:

- the robot model shows the estimated robot pose
- colored points show live LiDAR measurements
- the gray/white/black grid shows the generated occupancy map
- the map updates as the robot moves

The visualization was not perfectly smooth because RViz was running through VNC. This is expected because RViz renders a 3D scene with robot models, scan data, map layers, and transforms.

---

## 10. First Mapping Attempt

The first mapping attempt produced a larger map, but it was noisy and not selected as the final result.

![First saved map](Screenshots/2_Mapping/4_First_map.png)

The first attempt was still useful because it confirmed that the SLAM pipeline was working:

- LiDAR data was being published.
- RViz displayed the map.
- The robot pose was visible.
- The occupancy grid was generated and saved.
- The result showed the effect of unstable movement and noisy localization.

This map was not selected as the final result because the output contained too much noise and overlapping map fragments.

---

## 11. Saving the Final Map

After the second, more controlled room mapping run, I saved the map using `map_saver_cli`.

The selected final map name was:

```text
mentorpi_map_02
```

The command used:

```bash
ros2 run nav2_map_server map_saver_cli -f mentorpi_map_02 --ros-args -p map_subscribe_transient_local:=true
```

The terminal confirmed that the map was saved:

```text
Writing map occupancy data to mentorpi_map_02.pgm
Writing map metadata to mentorpi_map_02.yaml
Map saved
Map saved successfully
```

The saved map files were:

```text
mentorpi_map_02.pgm
mentorpi_map_02.yaml
```

For the repository, I placed them in:

```text
maps/mentorpi_map_02.pgm
maps/mentorpi_map_02.yaml
```

---

## 12. Final Saved Map Result

The final selected map is the second one-room mapping attempt.

![Final saved occupancy grid map](Screenshots/2_Mapping/5_Final_map.png)

The map is smaller than the first attempt, but it is the cleaner and more controlled result. It confirms that the LiDAR SLAM workflow works on the MentorPi T1.

This map is useful as a documented SLAM result. Further testing is still needed before using it for reliable autonomous navigation.

---

## 13. First Navigation Check

After saving the map, I started checking how it could be used for navigation.

The Hiwonder navigation lesson uses a map name parameter when launching navigation. For my saved map, the command would be:

```bash
ros2 launch navigation navigation.launch.py map:=mentorpi_map_02
```

RViz navigation visualization can be launched with:

```bash
ros2 launch navigation rviz_navigation.launch.py
```

During the first navigation check, the robot appeared on the saved map, but the localization was not fully stable. The robot pose appeared to jump on the map.

This means that the navigation stage was not completed yet.

The likely reasons are:

- the saved map is small
- the map still contains noise
- the robot’s initial pose was not set accurately
- the robot was not physically placed in the same position and direction as during mapping
- the LiDAR scan did not match the saved map strongly enough

Because of that, I stopped at this stage and did not mark autonomous navigation as completed.

---

## 14. Problems Encountered

### RViz over VNC was slow

RViz worked through VNC, but the visualization was not always smooth.

This is expected because RViz renders a graphical scene with robot models, LiDAR scan data, map layers, and coordinate transforms. VNC adds additional latency.

### The first map was noisy

The first mapping run produced a larger map, but it was noisy and not suitable as the final result.

Possible reasons:

- the robot moved too fast
- the robot turned too sharply
- the tank tracks slipped
- the route was too large for the first attempt
- the battery level may have affected movement stability

For this reason, I repeated the mapping process in a smaller area.

### The robot started beeping

During one of the mapping sessions, the robot started beeping. I stopped the test and placed it on charge.

Most likely causes:

- low battery
- power drop under load
- LiDAR, SLAM, RViz, VNC, and motors running at the same time
- controller warning state

For future SLAM and navigation tests, the robot should start with a fully charged battery.

### Navigation localization was unstable

The robot appeared on the saved map, but its pose was jumping in RViz.

This is likely a localization issue. The robot needs a good initial pose estimate and a map that matches the real environment closely.

The next navigation test should start with:

```text
1. Place the robot in the mapped room.
2. Launch navigation with mentorpi_map_02.
3. Use 2D Pose Estimate in RViz.
4. Set the robot’s initial position and direction carefully.
5. Wait until the pose stabilizes.
6. Set a very close navigation goal first.
```

---

## 15. What I Learned

In this stage, I learned how to:

- launch LiDAR-based SLAM on a ROS2 robot
- use RViz to visualize mapping
- use keyboard teleoperation during mapping
- understand the role of `/scan_raw`, `/map`, `/odom`, and `/tf`
- save an occupancy grid map with `map_saver_cli`
- compare mapping attempts and select the better result
- prepare a saved map for future navigation
- identify localization problems in RViz
- understand why slow and stable movement is important for mapping
- document a robotics mapping workflow for a portfolio project

The most important result is that I successfully generated and saved a real LiDAR-based occupancy grid map with the MentorPi T1.

---

## 16. Current Status

Completed:

```text
Remote access: done
Motion control: done
Keyboard teleoperation: done
LiDAR SLAM launch: done
RViz mapping visualization: done
First map generated: done
Second map generated: done
Final selected map: mentorpi_map_02
Map saving: done
```

Not completed yet:

```text
Stable localization: not yet
Autonomous route following: not yet
Reliable navigation goal execution: not yet
```

Next steps:

```text
1. Place the robot in the mapped room.
2. Launch navigation with mentorpi_map_02.
3. Set the initial pose carefully with 2D Pose Estimate.
4. Wait for localization to stabilize.
5. Send a very short navigation goal.
6. Record a successful autonomous navigation demo.
7. Add the navigation result as the next project stage.
```