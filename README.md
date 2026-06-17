# RCC Cohort Program Summer 2026 Project Definition

**Status:** 🚧 In Development

This is a living document. Update it as the team reaches decisions, and continue to update as the project evolves during development. Detailed documentation (architecture diagrams, system specs) should live in separate files in this repo.

**Program:** Robotics Career Coach (RCC) Cohort Program Summer 2026
**Contact:** [cohort@roboticscareercoach.com](mailto:cohort@roboticscareercoach.com)

-----

## Project Summary

This project is a mobile manipulation system built on [ROS 2](https://docs.ros.org) — a wheeled robot base equipped with a robotic arm, LiDAR, and an RGB-D camera that autonomously navigates to target locations in a warehouse-style environment and performs pick-and-place operations. The team will develop and validate the system in simulation first, then deploy it on a physical [Yahboom ROSMASTER M3 Pro](https://www.yahboom.net) robot owned by a team member to demonstrate sim-to-real transfer. The project gives each participant hands-on experience across the full robotics stack: SLAM, autonomous navigation ([Nav2](https://navigation.ros.org)), object detection ([YOLOv8n](https://docs.ultralytics.com)), arm control ([MoveIt 2](https://moveit.ros.org)), and data telemetry — while producing a portfolio artifact demonstrable to employers.

-----

## Demo Scenario

The operator launches a simulated warehouse environment with the robot at a known starting position and a target object placed at a fixed location. The robot localizes itself using a pre-built SLAM map, then [Nav2](https://navigation.ros.org) plans a collision-free path to the object's location. As the robot approaches, its RGB-D camera runs [YOLOv8n](https://docs.ultralytics.com) to detect and localize the object in 3D. The mounted arm executes a pick motion, grasps the object, and places it at a designated drop-off zone. Throughout the run, a live telemetry dashboard displays robot pose, planned path, LiDAR scan, and arm joint states. The demo concludes with the object placed at the destination and a telemetry summary.

The same scenario is then run on the physical [Yahboom ROSMASTER M3 Pro](https://www.yahboom.net) to validate sim-to-real transfer.

-----

## Tech Stack

> Open decisions are marked **[OPEN]**. These must be resolved in Week 1 before parallel workstreams begin. See [ARCHITECTURE.md](ARCHITECTURE.md) for rationale and trade-offs.

| Layer | Tool / Framework | Notes |
|---|---|---|
| Simulation | **[OPEN]** [Gazebo Harmonic](https://gazebosim.org/docs/harmonic) or [MuJoCo](https://mujoco.org) / [RoboSuite](https://robosuite.ai) | Isaac Sim excluded (requires RTX 3070+); see ARCHITECTURE.md |
| Middleware | [ROS 2](https://docs.ros.org) | Exact distro TBD — must be compatible with Yahboom M3 Pro |
| SLAM | **[OPEN]** [SLAM Toolbox](https://github.com/SteveMacenski/slam_toolbox) or [LIO-SAM](https://github.com/TixiaoShan/LIO-SAM) | LIO-SAM requires LiDAR + IMU; Alexander to confirm M3 Pro IMU in Week 1 |
| Navigation | [Nav2](https://navigation.ros.org) | |
| Manipulation | [MoveIt 2](https://moveit.ros.org) | |
| Perception | [YOLOv8n](https://docs.ultralytics.com) (Ultralytics) | CPU-viable; runs on low-spec hardware |
| Telemetry — live | [RViz2](https://github.com/ros2/rviz) | |
| Telemetry — pipeline | **[OPEN]** [Kafka](https://kafka.apache.org) + [Spark](https://spark.apache.org) + [Power BI](https://powerbi.microsoft.com) / custom | Durraiyah's ownership area; scope TBD in Week 1 |
| Real hardware | [Yahboom ROSMASTER M3 Pro](https://www.yahboom.net) | Alexander's robot; sim-to-real transfer targeted for Weeks 7–8 |
| Dev environment | [Docker](https://www.docker.com) + [GitHub Codespaces](https://github.com/features/codespaces) / [The Construct](https://www.theconstructsim.com) | Codespaces for low-spec members (120 core-hrs/month free tier) |
| CI | [GitHub Actions](https://github.com/features/actions) | |

-----

*Questions? Contact [cohort@roboticscareercoach.com](mailto:cohort@roboticscareercoach.com).*
