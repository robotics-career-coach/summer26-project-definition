# System Architecture

This document describes the intended system architecture for the RCC Summer 2026 mobile manipulation project. Update it via PR as open decisions are resolved.

---

## System Overview

A ROS 2-based mobile manipulation robot that:

1. Localizes itself and navigates autonomously to a target location (SLAM + Nav2)
2. Detects and localizes a target object using its RGB-D camera and YOLOv8n
3. Picks and places the object using a mounted robotic arm (MoveIt 2)
4. Streams telemetry data to a live dashboard throughout

The system is developed and validated in simulation first. Sim-to-real transfer to a physical Yahboom ROSMASTER M3 Pro is targeted for Weeks 7–8.

---

## Component Diagram

```mermaid
flowchart TD
    subgraph Robot["Robot — Sim or Real"]
        LiDAR[LiDAR]
        RGBD[RGB-D Camera]
        SLAM["SLAM<br/>(Toolbox / LIO-SAM)"]
        Perception["Perception<br/>(YOLOv8n + depth)"]
        Nav2["Nav2 Stack<br/>(planner · costmaps · controller)"]
        Base[Mobile Base Controller]
        Arm["Arm Controller<br/>(MoveIt 2)"]
    end

    Telemetry["Telemetry Layer<br/>(RViz2 · structured logging)"]

    LiDAR --> SLAM
    RGBD --> Perception
    SLAM -- "map + pose" --> Nav2
    Perception -- "/object_pose" --> Arm
    Nav2 -- "/cmd_vel" --> Base
    Base -- "/odom" --> Telemetry
    Nav2 -- "path + costmap" --> Telemetry
    Arm -- "joint states" --> Telemetry
    Perception -- "/detections" --> Telemetry
```

---

## Components

### Robot Platform

- **Base:** Differential-drive wheeled base (Yahboom ROSMASTER M3 Pro or sim equivalent)
- **Arm:** Robotic arm mounted on base (DOF from M3 Pro URDF)
- **Sensors:**
  - 2D or 3D LiDAR (SLAM + obstacle avoidance costmap)
  - RGB-D camera (object detection + depth projection)
  - IMU: Confirmed present on M3 Pro (Alexander verified)

---

### Simulation

**[OPEN: Gazebo Harmonic vs. MuJoCo / RoboSuite]**

| Option | Pros | Cons |
|---|---|---|
| Gazebo Harmonic | Native ROS 2 integration; Nav2 + MoveIt 2 tooling works out of the box; Basavaraj has direct experience | Less mature imitation learning ecosystem |
| MuJoCo / RoboSuite | Strong fit for imitation learning (RoboMimic); Alexander's proposed stack | Requires significant glue code to integrate Nav2; adds complexity |

**Recommendation:** Use Gazebo Harmonic as the primary simulation environment — it has the best compatibility with Nav2, MoveIt 2, and the full ROS 2 toolchain. If the team wants to pursue imitation learning (behavior cloning via RoboMimic), scope it as a parallel workstream with MuJoCo that shares only the perception code. Do not try to run both in the same simulation loop.

**Isaac Sim interop:** Isaac Sim includes a [MuJoCo (MJCF) importer extension](https://docs.isaacsim.omniverse.nvidia.com/6.0.0/importer_exporter/ext_isaacsim_asset_importer_mjcf.html) that can import MuJoCo XML models directly. This is an option if the team wants to evaluate Isaac Sim as a simulation environment while reusing MuJoCo assets from Menagerie.

---

### SLAM

**[OPEN: SLAM Toolbox vs. LIO-SAM]**

| Option | Pros | Cons | Requirement |
|---|---|---|---|
| SLAM Toolbox | ROS 2 native; well-supported; simpler setup; works in sim and on hardware | LiDAR-only (no IMU fusion) | LiDAR only |
| LIO-SAM | Tightly-coupled LiDAR + IMU fusion; robust in dynamic environments | More complex setup; needs a calibrated IMU | LiDAR + IMU |

**Recommendation:** Default to SLAM Toolbox. LIO-SAM is not necessary for an indoor warehouse environment, and the added complexity risks blocking Week 3. However, since Alexander has confirmed the M3 Pro has a working IMU, LIO-SAM is now a viable option if the team wants more robust localization for sim-to-real.

**Action item (Week 1):** ~~Alexander confirms M3 Pro IMU~~ ✓ Confirmed — Basavaraj updates SLAM choice → team closes this decision in the kickoff standup.

---

### Navigation

- **Nav2** full stack
- Costmap layers: static map layer + obstacle layer (from LiDAR)
- Global planner: NavFn or Smac Hybrid-A* (tune for warehouse environment)
- Local planner: DWB or MPPI (MPPI preferred for smoother trajectories near obstacles)
- Operator sends a `geometry_msgs/PoseStamped` goal; Nav2 executes

---

### Perception

1. YOLOv8n inference on RGB color image → 2D bounding boxes (`/detections`)
2. Bounding box center projected onto aligned depth image → 3D point in camera frame
3. TF transform to map frame → `/object_pose` (`geometry_msgs/PoseStamped`)

Notes:
- YOLOv8n is CPU-viable (nano model); no GPU required
- For the sim demo, train or use a pretrained model on a single object class (e.g., a box or can)
- Confidence threshold and NMS parameters will need tuning per environment

---

### Manipulation

- **MoveIt 2** motion planning
- Grasp pose computed from 3D object centroid + fixed offset (simplified; no full grasp pose estimation)
- Motion sequence: home → pre-grasp → grasp (close gripper) → lift → transport → place → open gripper → home
- Collision objects for the environment loaded into MoveIt 2 planning scene from the Nav2 costmap or manually

---

### Telemetry

**Minimum (all members):**
- RViz2: live robot pose, LiDAR scan, costmap, planned path, arm joint states, detection markers

**Structured logging [Durraiyah owns]:**
- ROS 2 topic subscribers (Python / rclpy) → structured log files (CSV / JSON)
- Candidate topics to log: `/odom`, `/amcl_pose`, `/scan`, `/joint_states`, `/detections`, navigation events

Durraiyah delivers structured logging of robot pose and detection events by Week 4.

---

## Open Technical Decisions

| Decision | Options | Blocking? | Decision Owner |
|---|---|---|---|
| Simulator | Gazebo Harmonic vs. MuJoCo/RoboSuite | Yes — blocks Weeks 2+ | Alexander + Basavaraj |
| SLAM algorithm | SLAM Toolbox vs. LIO-SAM | Yes — blocks Week 3 | Basavaraj (IMU confirmed ✓ — ready to decide) |
| Sim-to-real scope | Planned for Weeks 7–8; exact acceptance criteria TBD | No — in scope | Nick |
| Imitation learning | In scope vs. dropped | No — affects Alexander Weeks 1–4 | Alexander |

---

## Sim-to-Real Strategy

Physical target: **Yahboom ROSMASTER M3 Pro** (Alexander's hardware)
- Has LiDAR, RGB-D camera, robotic arm, and ROS 2 support
- URDF available from manufacturer (or extractable from manufacturer ROS package)

Transfer steps:
1. Develop URDF for simulation using the M3 Pro's exact dimensions and joint layout from day one — this minimizes the transfer gap
2. Validate all software stacks in simulation through Week 7
3. Week 7 (stretch): bring up ROS 2 on physical M3 Pro, verify sensor topics match URDF expectations
4. Retune Nav2 costmap and planner parameters for physical odometry noise and LiDAR characteristics
5. Retune YOLOv8n confidence threshold for real lighting conditions
6. Week 8 (stretch): live demo on hardware

The simulation demo is the **primary deliverable**. Sim-to-real transfer is planned for Weeks 7–8 and will be re-scoped if simulation milestones slip.

---

# ROSMASTER M3 Pro Hardware Interface

## Hardware Architecture

Physical components and the driver nodes that own them. Most peripherals route
through the STM32 controller, which `/YB_Node` talks to over serial.

```mermaid
flowchart TB
    subgraph HW["Physical Hardware"]
        BASE[Mecanum Drive Base]
        ARM[6-DOF Arm]
        L0[LiDAR 0]
        L1[LiDAR 1]
        IMU[IMU]
        BAT[Battery]
        BUZZ[Buzzer]
        LED[RGB LED]
        CAM[RGB Camera]
        JOY[Game Controller]
    end

    MCU[STM32 Controller]

    BASE --- MCU
    ARM --- MCU
    IMU --- MCU
    BAT --- MCU
    BUZZ --- MCU
    LED --- MCU

    MCU --- YB(["/YB_Node"])
    L0 --- YB
    L1 --- YB
    CAM --- CamDrv([Camera Driver])
    JOY --- JoyDrv(["/joy_node"])
```

## ROS 2 Node Graph

Following `rqt_graph` conventions: **ellipses are nodes**, **rectangles are topics**.
Arrows point from publisher to topic, and from topic to subscriber.

```mermaid
flowchart LR
    %% Nodes (ellipses)
    joy_node(["/joy_node"])
    joy_ctrl(["/joy_ctrl"])
    yb(["/YB_Node"])
    autostart(["/autostart_node"])

    %% Command flow
    joy_node --> joy["/joy"] --> joy_ctrl

    joy_ctrl --> cmd_vel["/cmd_vel"]
    autostart --> cmd_vel
    cmd_vel --> yb

    joy_ctrl --> arm_joint["/arm_joint"] --> yb
    joy_ctrl --> arm6["/arm6_joints"] --> yb
    joy_ctrl --> beep["/beep"] --> yb
    joy_ctrl --> rgb["/rgb"] --> yb

    joy_ctrl --> joystate["/JoyState"]
    joy_ctrl --> cancel["/move_base/cancel"]

    feedback["/joy/set_feedback"] --> joy_node

    %% Telemetry out of the hardware node
    yb --> battery["/battery"]
    yb --> imu["/imu/data_raw"]
    yb --> odom["/odom_raw"]
    yb --> scan0["/scan0"]
    yb --> scan1["/scan1"]
```

## Node Reference

| ROS Node | Description | Responsibility |
|----------|-------------|----------------|
| **`/YB_Node`** | Yahboom Hardware Interface Node | Primary interface between ROS 2 and the ROSMASTER hardware. Receives commands for the mobile base, robotic arm, buzzer, and RGB LED, and publishes hardware telemetry including LiDAR, IMU, odometry, and battery status. |
| **`/autostart_node`** | Autostart Node | Initializes the robot during startup and publishes initialization commands (including `/cmd_vel`) required during system bring-up. |
| **`/joy_node`** | Joystick Driver Node | Standard ROS 2 joystick driver. Reads the physical game controller and publishes `sensor_msgs/msg/Joy` messages. Accepts haptic feedback through `/joy/set_feedback`. |
| **`/joy_ctrl`** | Joystick Control Node | Converts joystick input into robot control commands. Publishes velocity commands (`/cmd_vel`), arm commands (`/arm_joint`, `/arm6_joints`), RGB LED commands (`/rgb`), buzzer commands (`/beep`), and joystick state (`/JoyState`). |

## Topic Reference

| Topic               | Message Type                        | Purpose                               |
| ------------------- | ----------------------------------- | ------------------------------------- |
| `/cmd_vel`          | `geometry_msgs/msg/Twist`           | Velocity commands to the mecanum base |
| `/odom_raw`         | `nav_msgs/msg/Odometry`             | Raw wheel odometry                    |
| `/imu/data_raw`     | `sensor_msgs/msg/Imu`               | IMU measurements                      |
| `/scan0`            | `sensor_msgs/msg/LaserScan`         | Primary LiDAR scan                    |
| `/scan1`            | `sensor_msgs/msg/LaserScan`         | Secondary LiDAR scan                  |
| `/arm_joint`        | `arm_msgs/msg/ArmJoint`             | Individual arm joint command          |
| `/arm6_joints`      | `arm_msgs/msg/ArmJoints`            | Six-joint arm state/command           |
| `/battery`          | `std_msgs/msg/Float32`              | Battery voltage or charge level       |
| `/joy`              | `sensor_msgs/msg/Joy`               | Joystick input                        |
| `/JoyState`         | `std_msgs/msg/Bool`                 | Joystick enable/status                |
| `/joy/set_feedback` | `sensor_msgs/msg/JoyFeedback`       | Controller vibration/feedback         |
| `/beep`             | `std_msgs/msg/UInt16`               | Buzzer command                        |
| `/rgb`              | `std_msgs/msg/ColorRGBA`            | RGB LED control                       |
| `/move_base/cancel` | `actionlib_msgs/msg/GoalID`         | Cancel navigation goal                |
| `/parameter_events` | `rcl_interfaces/msg/ParameterEvent` | ROS 2 parameter updates               |
| `/rosout`           | `rcl_interfaces/msg/Log`            | ROS logging                           |



*Open a PR to update this document when decisions are made. Tag the relevant team member as reviewer.*
