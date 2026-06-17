# 8-Week Execution Plan

**Program:** RCC Cohort Program Summer 2026
**Updated:** 2026-06-16

> Open decisions (simulator, SLAM algorithm, sim-to-real scope) must be resolved in Week 1. Items marked **[OPEN]** cannot be fully specified until those choices are made.

---

## Parallel Workstream Strategy

After the shared Week 2 foundation, work splits into three independent tracks that run in parallel through Weeks 3–4 and converge at integration in Week 6.

```mermaid
gantt
    dateFormat  WW
    axisFormat  Week %W

    section All
    Kickoff & setup          : w01, 1w
    Shared foundation        : w02, 1w

    section Navigation
    SLAM + mapping           : w03, 1w
    Nav2 + waypoints         : w04, 1w
    Nav validation           : w05, 1w

    section Manipulation
    MoveIt 2 + fixed base    : w03, 1w
    Pick from known pose     : w04, 1w
    Pick from detected pose  : w05, 1w

    section Vision
    YOLOv8n on test images   : w03, 1w
    Depth fusion + /object_pose : w04, 1w
    Vision validation        : w05, 1w

    section All
    Full pipeline integration : w06, 1w
    Polish & testing         : w07, 1w
    Demo & wrap-up           : w08, 1w
```

**Track owners:**
| Track | Lead | Supporting |
|---|---|---|
| Navigation | Basavaraj | Arjun (CI + interfaces) |
| Manipulation | Arjun | Basavaraj (MoveIt 2 config), Alexander (perception hand-off) |
| Vision | Alexander | Durraiyah (telemetry logging) |

The tracks are designed to be decoupled:
- **Manipulation** works with the base at a fixed, hardcoded pose — no Nav2 or SLAM needed.
- **Vision** works with a static camera position in sim or pre-captured test images — no navigation or arm needed.
- **Navigation** builds the mobile base stack independently and hands off to the full system at Week 6.

---

## Milestones at a Glance

| Week | Theme | Key Deliverable |
|---|---|---|
| 1 | Kickoff & setup | Dev environment running; all blocking decisions closed |
| 2 | Shared foundation | Robot URDF in sim; all sensor topics verified |
| 3 | Parallel tracks begin | Nav: map generated · Arm: MoveIt 2 moving joints · Vision: YOLOv8n detecting objects |
| 4 | Subsystems mature | Nav: autonomous waypoints · Arm: picks from hardcoded pose · Vision: 3D pose published |
| 5 | Cross-track integration | Arm picks from camera-detected pose; Nav validated independently |
| 6 | Full pipeline | Navigate → detect → pick → place (2/3 runs succeed) |
| 7 | Polish & testing | All acceptance criteria met; dry-run demo recorded |
| 8 | Demo & wrap-up | Demo video published; repo documented and clean |

---

## Week-by-Week Detail

### Week 1 — Kickoff & Environment Setup

**Goal:** Every team member can build and run the project; all blocking decisions are made.

| Task | Owner |
|---|---|
| Host kickoff standup; decide on simulator, SLAM, sim-to-real scope, telemetry scope | Nick |
| Confirm M3 Pro IMU presence/absence (unlocks SLAM decision) | Alexander |
| Create shared Docker image + `docker-compose.yml`; verify it builds on GitHub Codespaces | Arjun |
| Confirm chosen simulator launches in Docker container | Basavaraj |
| Complete ROS 2 rclpy beginner tutorial (pub/sub); write hello-world subscriber node | Durraiyah |
| Assign testing and acceptance criteria ownership across the team | Nick |
| Open PR updating README.md and ARCHITECTURE.md with resolved decisions | All |

**Done when:** All four members have the dev container running (locally or on Codespaces) and all blocking decisions are documented in a merged PR.

---

### Week 2 — Shared Foundation

**Goal:** The robot model is in the simulator with all sensor topics live. This is the shared baseline all three tracks depend on.

| Task | Owner |
|---|---|
| Build warehouse-style world file (walls, shelves, floor, obstacle objects) | Basavaraj |
| Source or build M3 Pro URDF; verify joint names and geometry | Alexander |
| Spawn URDF in sim; verify `/scan`, `/camera/color/image_raw`, `/camera/depth/image_raw` topics publish | Basavaraj |
| Confirm arm joint state topics (`/joint_states`) publish correctly | Basavaraj |
| Add CI job: ROS 2 workspace builds on every PR | Arjun |
| Subscribe to `/odom` and `/scan`; log timestamped CSV as first telemetry deliverable | Durraiyah |
| Manual test: drive robot via teleop; verify no clipping, collision, or physics issues | TBD |

**Done when:** Robot drives in sim via teleop with LiDAR, RGB-D, and arm joint state topics all visible in RViz2.

---

### Weeks 3–4 — Parallel Tracks

The three tracks below run concurrently. Each has its own definition of done and does not block the others.

---

#### Track A — Navigation (Basavaraj)

**Goal by end of Week 4:** Robot navigates autonomously to any waypoint in the sim environment without collision.

**Week 3 tasks:**
| Task | Owner |
|---|---|
| Integrate SLAM; teleop to build map; save `map.pgm` + `map.yaml` | Basavaraj |
| Launch map server + localization; verify pose estimate in RViz2 | Basavaraj |
| Add robot pose to telemetry log | Durraiyah |

**Week 4 tasks:**
| Task | Owner |
|---|---|
| Configure Nav2 (costmaps, global + local planners); integrate with saved map | Basavaraj |
| Write goal publisher node (goal x/y/θ → Nav2 action) | Arjun |
| Test navigation to 3 hardcoded waypoints; tune planner parameters | Basavaraj |
| Log navigation events (goal sent, path planned, reached/failed) to telemetry | Durraiyah |

**Track A done when:** Robot navigates to 3 different goal poses without collision, 3/3 trials.

---

#### Track B — Manipulation (Arjun + Basavaraj)

**Goal by end of Week 4:** Arm picks an object from a hardcoded, known pose and places it at a target pose — with the base fixed in position.

The base is either spawned at a fixed pose in sim or held in place with Nav2 goals disabled. No SLAM or localization is needed for this track.

**Week 3 tasks:**
| Task | Owner |
|---|---|
| Configure MoveIt 2 for the sim arm; verify joint states and planning scene | Basavaraj |
| Command arm to a set of test joint configurations; verify no self-collision | Arjun |

**Week 4 tasks:**
| Task | Owner |
|---|---|
| Write pick-and-place action server: hardcoded grasp pose → plan → execute pick → place | Arjun |
| Test pick sequence from 3 hardcoded object positions | Arjun |
| Log arm action events (goal pose, success/fail) to telemetry | Durraiyah |

**Track B done when:** Arm picks an object from each of 3 hardcoded poses and places it at a target, 3/3 trials.

---

#### Track C — Vision (Alexander + Durraiyah)

**Goal by end of Week 4:** YOLOv8n detects a target object and publishes its 3D pose in the map frame — using either the sim camera or static test images.

Development can start with pre-captured images or a static camera in sim; no navigation or arm movement is needed.

**Week 3 tasks:**
| Task | Owner |
|---|---|
| Write YOLOv8n ROS 2 node; publish `/detections` on RGB image | Alexander |
| Test detection against a set of static test images; tune confidence threshold | Alexander |
| Log detection events (class, confidence, bounding box) to telemetry | Durraiyah |

**Week 4 tasks:**
| Task | Owner |
|---|---|
| Depth projection: project detection center to 3D point in camera frame | Alexander |
| TF transform: publish `/object_pose` in map frame (`geometry_msgs/PoseStamped`) | Alexander |
| Validate `/object_pose` against known ground-truth object positions in sim | Alexander |
| Add 3D pose to telemetry log | Durraiyah |

**Track C done when:** Object placed at 5 known positions in sim; `/object_pose` error < 5 cm from ground truth, 4/5 trials.

---

### Week 5 — Cross-Track Integration

**Goal:** Vision output drives the arm; Navigation is independently validated. Each sub-system pairing is tested before the full three-way integration in Week 6.

| Task | Owner |
|---|---|
| Wire `/object_pose` (Track C) into pick-and-place action server (Track B); test arm picking from camera-detected pose | Alexander + Arjun |
| Navigation acceptance test: robot reaches 3 goal poses, 3/3, no collisions (Track A sign-off) | TBD |
| Perception acceptance test: detection rate ≥ 4/5 at known positions (Track C sign-off) | TBD |
| Manipulation acceptance test: pick from detected pose succeeds 3/3 (Track B + C sign-off) | TBD |
| Telemetry dashboard: live pose, path, detection markers, arm state visible in RViz2 | Durraiyah |

**Done when:** Vision → Arm integration succeeds 3/3 trials; Navigation validated independently 3/3 trials.

---

### Week 6 — Full Pipeline Integration

**Goal:** All three tracks integrated into a single end-to-end mission.

| Task | Owner |
|---|---|
| Wire mission executor: Nav2 drives base to object location → Vision detects → Arm picks and places | Arjun |
| Integration testing; file GitHub issues for each failure mode | All |
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
| Finalize telemetry dashboard; confirm it runs cleanly during a full mission | Durraiyah |
| Bring up ROS 2 on physical M3 Pro; verify sensor topics match URDF | Alexander |
| Record dry-run demo; review and identify rough edges | All |

**Done when:** ≥ 3/3 end-to-end runs succeed in sim; dry-run demo recorded.

---

### Week 8 — Final Demo & Wrap-Up

**Goal:** Deliverables complete; team has a portfolio artifact.

| Task | Owner |
|---|---|
| Record final demo video (simulation) | All |
| Live demo on M3 Pro hardware | Alexander |
| Finalize all repo documentation (README, ARCHITECTURE.md, ROLES.md, inline code comments) | All |
| Publish repo and distribute to participants for portfolio use | Nick |
| Program retrospective | All |

**Deliverables:**
- Recorded demo video (simulation and hardware)
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
| Arm pick-and-place too complex for timeline | Medium | Medium | Track B starts with hardcoded grasp poses — full grasp pose estimation is not required; re-scope in Week 4 standup if needed |
| Vision–Arm integration harder than expected | Medium | Medium | Week 5 is dedicated to this pairing; if blocked, fall back to hardcoded pose for demo and note it as future work |
| Sim-to-real transfer harder than expected | Medium | Medium | Simulation demo is the primary deliverable; if hardware bring-up slips, re-scope to partial hardware validation and document remaining gaps |
| Testing tasks unowned after team change | Medium | Medium | Assign testing ownership at Week 1 kickoff; distribute across existing members or recruit a replacement |
| Timezone conflicts slow async decisions | Medium | Medium | Blocking decisions made synchronously at kickoff; subsequent decisions via GitHub PRs with 48-hour review window |

---

*Questions? Contact [cohort@roboticscareercoach.com](mailto:cohort@roboticscareercoach.com).*
