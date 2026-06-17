# Roles and Responsibilities

This document assigns a **recommended** primary workstream to each team member based on their background. These are starting points, not rigid boundaries. Everyone is encouraged to contribute outside their area, pair with teammates on unfamiliar problems, and explore parts of the stack they want to learn. Primary ownership just means someone is accountable for that workstream moving forward — it doesn't mean others can't touch it.

Review and adjust at the Week 1 kickoff if the team has different preferences.

---

## Team

| Member | Location | Timezone | Role |
|---|---|---|---|
| Nick Kirsch | — | — | Program Manager |
| Alexander Mak | Philadelphia, PA | ET | Perception & Hardware Lead |
| Arjun Malarmannan | Boston, MA | ET | Software Engineering Lead |
| Basavaraj Chikki | Bengaluru, India | IST | Navigation & Simulation Lead |
| Durraiyah Muneer | Karachi, Pakistan | PKT | Telemetry & Data Lead |

---

## Workstream Ownership

### Nick Kirsch — Program Manager

**Role:** Provides guidance to the cohort, structure to the program, and holds the team accountable for deliverables and timelines.

**Primary ownership:**
- Kickoff facilitation and resolution of blocking decisions
- Weekly standup structure and cadence
- Testing and acceptance criteria ownership assignments
- Final repo publication and distribution to participants for portfolio use
- Program retrospective

**Not a developer on the project** — Nick does not own code workstreams but participates in PR reviews and architectural decisions as needed.

---

### Alexander Mak — Perception & Hardware Lead

**Background:** Python, C/C++, ROS 2, YOLO, TensorRT, Docker, AWS, Jetson experience; owns the Yahboom ROSMASTER M3 Pro

**Primary ownership:**
- YOLOv8n inference node (object detection on RGB image, published as `/detections`)
- 3D object localization (depth projection + TF transforms → `/object_pose`)
- Yahboom M3 Pro URDF — source or build; use as the sim model from Week 1
- Sim-to-real transfer: hardware bring-up, sensor calibration, parameter retuning

**Contributing to:**
- Simulation environment (robot model and sensor plugins, with Basavaraj)
- Arm perception integration (hand-off object pose to MoveIt 2 pipeline)

**Open item:** If imitation learning (behavior cloning via RoboMimic on MuJoCo) stays in scope, Alexander leads that as a parallel Milestone B. Confirm scope at kickoff.

---

### Arjun Malarmannan — Software Engineering Lead

**Background:** PLCs, AMR systems, JIRA, Azure DevOps, REST APIs; real warehouse robotics experience

**Primary ownership:**
- Mission executor node: wires Nav2 → perception → pick-and-place into a single coordinated flow
- Pick-and-place action server (MoveIt 2 integration)
- ROS 2 topic/service interface contracts (document what each node publishes/subscribes)

**Contributing to:**
- Navigation stack (Nav2 goal publisher, waypoint testing, with Basavaraj)
- Simulation environment setup and validation
- MoveIt 2 arm configuration and testing
- Shared Docker image + `docker-compose.yml`
- Testing coordination (define integration test structure; owner TBD at kickoff)

**Note:** Arjun's warehouse AMR experience is a valuable reality check — flag when simulation assumptions diverge from how real AMR systems behave. This role is intentionally broad to give Arjun hands-on experience across the full robotics stack.

---

### Basavaraj Chikki — Navigation & Simulation Lead

**Background:** C++, ROS 2, SLAM, Nav2, Gazebo, ESP32; has built warehouse navigation robots from scratch

**Primary ownership:**
- Simulation world (Gazebo Harmonic or chosen sim): warehouse environment, object placement, physics
- SLAM integration (SLAM Toolbox or LIO-SAM — see [ARCHITECTURE.md](ARCHITECTURE.md)); map generation workflow
- Nav2 stack configuration: costmaps, global planner, local planner, recovery behaviors
- Map save/serve pipeline (`map_saver_cli` → `map_server` → Nav2 lifecycle)

**Contributing to:**
- Robot URDF validation in sim (with Alexander)
- MoveIt 2 arm integration (joint state bridging)

---

### Durraiyah Muneer — Telemetry & Data Lead

**Background:** Python, Java, Kafka, Spark, AWS, Azure, Power BI; strong data engineering; no prior ROS 2 experience

**Primary ownership:**
- Telemetry pipeline design and implementation (scope finalized in Week 1)
- **Minimum deliverable:** Python rclpy subscriber nodes logging `/odom`, `/amcl_pose`, `/detections`, and navigation events to structured files
- **Full deliverable (if scoped in):** ROS 2 data bridge → Kafka → Spark Streaming → Power BI or custom dashboard
- Dashboard: robot pose history, path taken, detection events, arm action log, mission outcomes

**ROS 2 ramp-up path (Weeks 1–2):**
- Complete ROS 2 beginner Python tutorials (publisher/subscriber with rclpy)
- First task: write a subscriber node for `/odom` that prints and logs to CSV — this is pure Python, no C++ required
- The Kafka bridge is standard Kafka producer code once the ROS 2 subscriber is working

**Note:** Durraiyah's data engineering background is the team's strongest asset for the telemetry workstream. Kafka/Spark experience maps directly once the ROS 2 data bridge is in place.

---

## Shared Responsibilities

| Responsibility | Expectation |
|---|---|
| Weekly standup | Required for all; async written update acceptable as fallback |
| PR reviews | Every PR needs 1 approving review from a non-author before merge |
| GitHub issues | File an issue for blockers; assign to yourself; close when resolved |
| ROS 2 onboarding | Durraiyah: complete ROS 2 beginner tutorials by end of Week 1 |
| Apache 2.0 compliance | All code committed to the repo must be compatible with the Apache 2.0 license |

---

## Timezones

| Member | Timezone | UTC Offset (summer) |
|---|---|---|
| Alexander Mak | US Eastern (ET) | UTC−4 |
| Arjun Malarmannan | US Eastern (ET) | UTC−4 |
| Basavaraj Chikki | India Standard Time (IST) | UTC+5:30 |
| Durraiyah Muneer | Pakistan Standard Time (PKT) | UTC+5 |

The team is **async-first** due to the gap between US Eastern and South Asian timezones. GitHub comments and Slack are the primary collaboration channels. Blocking decisions should be made via GitHub PRs with a 48-hour review window so no one is forced into off-hours work. The weekly standup time should be set via a Doodle poll in Week 1.

---

*Questions? Contact [cohort@roboticscareercoach.com](mailto:cohort@roboticscareercoach.com).*
