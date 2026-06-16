# 8-Week Execution Plan

> Open decisions (simulator, SLAM algorithm, sim-to-real scope) must be resolved in Week 1. Items marked **[OPEN]** cannot be fully specified until those choices are made.

---

## Milestones at a Glance

| Week | Theme | Key Deliverable |
|---|---|---|
| 1 | Kickoff & setup | Dev environment running for all members; all blocking decisions closed |
| 2 | Robot in sim | Robot URDF spawned in simulator; sensor topics verified in RViz2 |
| 3 | SLAM & mapping | Map generated; robot localizes to within 0.3 m of ground truth |
| 4 | Autonomous navigation | Nav2 navigates to 3 goal waypoints without collision |
| 5 | Perception & arm | Object detected; arm picks and places from detected pose |
| 6 | Full pipeline | End-to-end: navigate → detect → pick → place (2/3 runs succeed) |
| 7 | Polish & testing | All acceptance criteria met; dry-run demo recorded |
| 8 | Final demo & wrap-up | Demo video published; repo documented and clean |

---

## Week-by-Week Detail

### Week 1 — Kickoff & Environment Setup

**Goal:** Every team member can build and run the project; all blocking decisions are made.

| Task | Owner |
|---|---|
| Host kickoff standup; facilitate decision on simulator, SLAM, sim-to-real scope, telemetry scope | Nick |
| Confirm M3 Pro IMU presence/absence (unlocks SLAM decision) | Alexander |
| Create shared Docker image + `docker-compose.yml`; verify it builds on GitHub Codespaces | Arjun |
| Confirm chosen simulator launches in Docker container | Basavaraj |
| Complete ROS 2 rclpy beginner tutorial (pub/sub); write hello-world subscriber node | Durraiyah |
| Assign testing and acceptance criteria ownership across the team | Nick |
| Open PR updating README.md and ARCHITECTURE.md with resolved decisions | All |

**Done when:** All four members have the dev container running (locally or on Codespaces) and all blocking decisions are documented in a merged PR.

---

### Week 2 — Robot Model & Simulation Environment

**Goal:** The robot is spawned in the simulator with working sensor topics.

| Task | Owner |
|---|---|
| Build warehouse-style world file (walls, shelves, floor, obstacle objects) | Basavaraj |
| Source or build M3 Pro URDF; verify joint names and geometry | Alexander |
| Spawn URDF in sim; verify `/scan`, `/camera/color/image_raw`, `/camera/depth/image_raw` topics publish | Basavaraj |
| Add CI job: ROS 2 workspace builds on every PR | Arjun |
| Subscribe to `/odom` and `/scan`; log timestamped CSV as first telemetry deliverable | Durraiyah |
| Manual test: drive robot via teleop, verify no clipping, collision, or physics issues | TBD |

**Done when:** Robot drives in sim via teleop with LiDAR and RGB-D topics visible and logging in RViz2.

---

### Week 3 — SLAM & Mapping

**Goal:** A persistent map of the simulated environment is generated; the robot localizes within it.

| Task | Owner |
|---|---|
| Integrate chosen SLAM algorithm; configure parameters for sim environment | Basavaraj |
| Teleop robot around warehouse world; save map (`map.pgm` + `map.yaml`) | Basavaraj |
| Launch map server + AMCL (or SLAM Toolbox localization); verify pose estimate in RViz2 | Basavaraj |
| Verify RGB-D camera field of view is not occluded by arm in URDF | Alexander |
| Add robot pose (`/amcl_pose` or `/slam_toolbox/pose`) to telemetry log | Durraiyah |
| Localization test: robot placed at 3 different known poses; verify AMCL converges within 0.3 m | TBD |

**Done when:** Robot spawns at a random pose and localizes within 0.3 m of ground truth within 30 seconds, verified across 3 poses.

---

### Week 4 — Autonomous Navigation

**Goal:** Nav2 navigates the robot to a goal pose, avoiding obstacles.

| Task | Owner |
|---|---|
| Configure Nav2 (costmaps, global planner, local planner); integrate with saved map and AMCL | Basavaraj |
| Test navigation to 3 hardcoded waypoints; tune planner parameters | Basavaraj |
| Write goal publisher node with a simple interface (goal x/y/theta → Nav2 action) | Arjun |
| Log navigation events (goal sent, path planned, goal reached/failed) to telemetry pipeline | Durraiyah |
| Navigation acceptance test: robot reaches goal within 0.2 m, no collisions, 3/3 trials | TBD |

**Done when:** Robot autonomously navigates to 3 different goal poses without collision, 3/3 trials.

---

### Week 5 — Perception & Arm Control

**Goal:** YOLOv8n detects a target object; the arm picks and places it.

| Task | Owner |
|---|---|
| Write YOLOv8n ROS 2 node; publish `/detections` (bounding boxes on RGB image) | Alexander |
| Depth projection: project detection center to 3D; publish `/object_pose` in map frame | Alexander |
| Configure MoveIt 2 for sim arm; verify joint states publish correctly | Basavaraj |
| Write pick-and-place action server: receives object pose → plans arm motion → executes | Arjun |
| Log detection events (class, confidence, 3D pose, timestamp) to telemetry pipeline | Durraiyah |
| Perception test: object placed at 5 known positions; detection rate ≥ 4/5 | TBD |

**Done when:** Arm successfully picks an object from a camera-detected pose and places it at a target pose, detection rate ≥ 4/5.

---

### Week 6 — Full Pipeline Integration

**Goal:** End-to-end mission: robot navigates to object, detects it, picks and places it.

| Task | Owner |
|---|---|
| Wire Nav2 → detection → pick-and-place into a single mission executor node | Arjun |
| Integration testing; file GitHub issues for each failure mode | All |
| Telemetry dashboard: live pose, path, detection events, arm state (RViz2 at minimum) | Durraiyah |
| End-to-end test: 3 full mission runs; record success rate and failure modes | TBD |
| Bug fixing sprint targeting blockers for the demo | All |

**Done when:** 2 out of 3 end-to-end mission runs succeed in sim.

---

### Week 7 — Polish, Testing & Demo Prep

**Goal:** All acceptance criteria met; demo is rehearsable and recordable.

| Task | Owner |
|---|---|
| Run full test suite against all acceptance criteria; produce pass/fail report | TBD |
| Address remaining test failures (triage in standup) | All |
| Finalize CI pipeline; all tests passing on `main` | Arjun |
| Finalize telemetry dashboard; ensure it runs without errors during a full mission | Durraiyah |
| *(Stretch)* Bring up ROS 2 on physical M3 Pro; verify sensor topics match URDF | Alexander |
| Record dry-run demo; review and identify rough edges | All |

**Done when:** ≥ 3/3 end-to-end runs succeed in sim; dry-run demo recorded.

---

### Week 8 — Final Demo & Wrap-Up

**Goal:** Deliverables complete; team has a portfolio artifact.

| Task | Owner |
|---|---|
| Record final demo video (simulation) | All |
| *(Stretch)* Live demo on M3 Pro hardware | Alexander |
| Finalize all repo documentation (README, ARCHITECTURE.md, ROLES.md, inline code comments) | All |
| Publish repo and distribute to participants for portfolio use | Nick |
| Program retrospective | All |

**Deliverables:**
- Recorded demo video (sim; stretch: hardware)
- Clean, documented GitHub repo (Apache 2.0)
- Architecture document, roles document, and execution plan updated to reflect what was actually built
- Final README with instructions for running the demo

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Simulator decision not resolved in Week 1 | Medium | High — blocks Week 2+ | Nick facilitates decision at kickoff; default to Gazebo Harmonic if no consensus by end of Week 1 |
| Low-spec hardware can't run sim locally | High | Medium | GitHub Codespaces as primary fallback; Arjun validates Docker image on Codespaces in Week 1 |
| Durraiyah's ROS 2 ramp-up takes > 2 weeks | Medium | Low | Telemetry starts with pure-Python CSV logging (no ROS 2 needed until Week 3); Kafka work begins in parallel |
| M3 Pro has no IMU → LIO-SAM not viable | Medium | Low | Default to SLAM Toolbox from the start; decision closes once Alexander confirms IMU in Week 1 |
| Arm pick-and-place too complex for timeline | Medium | Medium | Use fixed grasp offset from detected centroid — skip full grasp pose estimation; re-scope in Week 5 standup if needed |
| Sim-to-real fails in Weeks 7–8 | Medium | Low | Sim demo is the primary deliverable; hardware is explicitly a stretch goal and will be cut without affecting program success |
| Testing tasks unowned after team change | Medium | Medium | Assign testing ownership at Week 1 kickoff; distribute across existing members or recruit a replacement |
| Timezone conflicts slow async decisions | Medium | Medium | All blocking decisions made synchronously at kickoff; subsequent decisions via GitHub PRs with 48-hour review window |

---

*Questions? Contact [cohort@roboticscareercoach.com](mailto:cohort@roboticscareercoach.com).*
