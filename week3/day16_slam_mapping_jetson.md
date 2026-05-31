# Day 16 — 실제 SLAM 매핑 (Jetson 온보드 실행)

> **목표:** Jetson Orin Nano에서 직접 slam_toolbox를 실행하여 환경 지도를 생성하고 저장한다.

---

## 16.1 실행 구조: Remote PC 불필요

RPi 환경에서는 SLAM을 Remote PC에서 실행했지만, Jetson에서는 **모든 것을 온보드에서 실행**합니다.

```
┌──────────────────────────────────────────────────┐
│                 Jetson Orin Nano                 │
│                                                  │
│  ┌──────────┐  /scan ┌──────────────┐  /map  ┌──┴──┐
│  │Bringup   │───────►│ SLAM Toolbox │───────►│RViz2│
│  │(LIDAR+   │ /odom  │  (onboard)   │        │     │
│  │ OpenCR)  │───────►│              │        │GUI  │
│  └──────────┘        └──────┬───────┘        └─────┘
│                             │
│                    ┌────────▼───────┐
│                    │ Teleop Keyboard │
│                    └────────────────┘
└──────────────────────────────────────────────────┘
```

---

## 16.2 준비

```bash
# 1. TurtleBot3 Bringup
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_bringup robot.launch.py

# 2. 새 터미널에서 SLAM Toolbox 실행
ros2 launch slam_toolbox online_async_launch.py \
  slam_params_file:=/opt/ros/humble/share/turtlebot3_navigation2/param/burger.yaml \
  use_sim_time:=false
```

---

## 16.3 RViz2 온보드 실행

```bash
# 3. 새 터미널에서 RViz2 실행 (Jetson 모니터에서 직접!)
rviz2
```

**RViz2 설정:**
1. Fixed Frame → `map`
2. Add → `Map` → Topic: `/map`
3. Add → `LaserScan` → Topic: `/scan`
4. Add → `RobotModel`
5. 저장: `File → Save Config` → `~/jetson_slam.rviz`

---

## 16.4 Teleop으로 매핑

```bash
# 4. 새 터미널에서 텔레옵
ros2 run turtlebot3_teleop teleop_keyboard
```

---

## 16.5 지도 저장

```bash
# 5. 매핑 완료 후 저장
ros2 run nav2_map_server map_saver_cli -f ~/jetson_map

# 확인
ls -la ~/jetson_map.yaml ~/jetson_map.pgm
```

---

## 16.6 GPU 가속 매핑 성능

Jetson Orin Nano에서 SLAM 실행 시 GPU 자원 사용:

```bash
# jtop으로 SLAM + RViz2 동시 GPU 사용률 확인
jtop

# 예상: SLAM 10-20% + RViz2 5-15% GPU
# 여유 GPU로 추가 AI 작업 가능
```

---

## 📝 연습 문제

> RPi 버전 Day 16과 동일 (생략)
> 추가: SLAM 실행 중 GPU 사용률을 jtop으로 기록하고 CPU 전용 대비 성능 비교
