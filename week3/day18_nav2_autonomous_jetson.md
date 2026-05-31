# Day 18 — Nav2 자율주행 (Jetson 온보드 실행)

> **목표:** Jetson Orin Nano에서 직접 Navigation2를 실행하여 자율주행을 구현한다.

---

## 18.1 실행 순서 (모두 Jetson 온보드)

### 터미널 1: Bringup

```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_bringup robot.launch.py
```

### 터미널 2: SLAM 지도 로드 또는 SLAM 실행

```bash
# 옵션 A: 저장된 지도로 Nav2 실행
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
  map:=~/jetson_map.yaml \
  use_sim_time:=false

# 옵션 B: SLAM을 먼저 실행 (Day 16)
ros2 launch slam_toolbox online_async_launch.py \
  slam_params_file:=/opt/ros/humble/share/turtlebot3_navigation2/param/burger.yaml \
  use_sim_time:=false
```

### 터미널 3: RViz2

```bash
rviz2
```

---

## 18.2 온보드 Nav2 작업 흐름

```
1. RViz2 → "2D Pose Estimate" → 지도에 초기 위치 설정
2. RViz2 → "2D Nav Goal" → 목적지 설정
3. Nav2가 경로 계획 및 추종 시작
4. AMCL이 실시간 위치 추정
5. 목적지 도착 확인
```

---

## 18.3 터미널 관리 (tmux)

Jetson 단일 시스템에서 3-4개 터미널을 효율적으로 관리:

```bash
# tmux 세션 생성
tmux new -s nav2_session

# 레이아웃: 2x2 분할
Ctrl+B  →  :select-layout tiled

# 각 창 할당:
# 창 0: Bringup
# 창 1: Nav2 launch
# 창 2: RViz2 (GUI이므로 외부)
# 창 3: 모니터링 (ros2 topic echo /patrol_status 등)
```

---

## 18.4 성능 모니터링

Nav2 자율주행 중 Jetson 리소스 사용량:

```bash
# GPU 사용률
jtop

# CPU 코어별 사용률
htop

# 메모리
free -h

# 예상 사용량 (Nav2 + RViz2 + Bringup 동시):
# CPU: 30~50%
# GPU: 15~25%
# RAM: 3~5GB
```

---

## 18.5 문제 발견 시

> 상세한 Nav2 파라미터 튜닝은 **RPi 버전 Day 18**을 참조하세요.
> Jetson은 성능이 충분하므로 기본 파라미터로도 대부분 잘 동작합니다.

---

## 📝 연습 문제

> RPi 버전 Day 18과 동일
> 추가: Nav2 자율주행 중 Jetson의 GPU/CPU 사용률을 측정하고 RPi+PC 조합과 성능 비교
