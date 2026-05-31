# Day 3 — Remote PC 세팅 (선택 사항)

> **목표:** 필요 시 Remote PC에 ROS2 Humble Desktop을 설치하고 Jetson과 통신을 확인한다.

---

## 3.1 Remote PC가 필요한 경우

Jetson Orin Nano는 자체 성능이 충분하므로 **Remote PC 없이도 모든 작업이 가능**합니다.
다음 상황에서만 Remote PC를 준비하세요:

1. **Jetson에 모니터/키보드 연결이 어려운 경우** (헤드리스 운영)
2. **여러 터미널이 필요한 복잡한 작업** (tmux로 대체 가능)
3. **GUI 도구를 더 큰 화면에서 사용하고 싶은 경우**

> **권장:** Jetson Orin Nano에 직접 모니터 연결 + tmux 사용 = Remote PC 불필요

---

## 3.2 Remote PC 준비 (필요 시)

RPi 버전 Day 3와 **완전히 동일**한 내용입니다.

```bash
# 참고: RPi 버전 자료
# ../turtlebot3-ros2-curriculum/week1/day03_remote_pc_setup.md
```

### 요약 명령어 (Remote PC Ubuntu 22.04)

```bash
# ROS2 Desktop 설치
sudo apt install -y ros-humble-desktop
echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
source ~/.bashrc

# Gazebo
sudo apt install -y ros-humble-gazebo-*

# TurtleBot3 시뮬레이션
sudo apt install -y ros-humble-turtlebot3*

# ROS_DOMAIN_ID (Jetson과 동일하게)
echo 'export ROS_DOMAIN_ID=30' >> ~/.bashrc
```

### 통신 확인

```bash
# Jetson에서
ros2 run demo_nodes_py talker &

# PC에서
ros2 topic list
ros2 topic echo /chatter
```

---

## 📝 연습 문제

> RPi 버전 Day 3의 연습 문제와 동일합니다.
