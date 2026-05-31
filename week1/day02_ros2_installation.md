# Day 2 — Jetson Orin Nano에 ROS2 Humble 설치

> **목표:** Jetson에 ROS2 Humble Hawksbill (ros-base)을 설치하고 정상 동작을 확인한다.

---

## 2.1 개요

Jetson Orin Nano는 Ubuntu 22.04 LTS (aarch64)를 사용하므로, ROS2 Humble을 **apt 패키지로 직접 설치**할 수 있습니다.

> **RPi 버전과의 차이점:** 설치 과정이 거의 동일합니다. Jetson은 성능이 충분하므로 `ros-base` 대신 `ros-humble-desktop`을 설치해도 무방합니다.

---

## 2.2 Locale 설정

```bash
sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
locale
```

---

## 2.3 ROS2 Repository 추가

```bash
# GPG 키 추가
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

# Repository 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu jammy main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
```

---

## 2.4 ROS2 Humble 설치

Jetson Orin Nano의 충분한 성능을 고려하여 **Desktop 버전**(GUI 도구 포함) 설치를 권장합니다:

```bash
# Option A: Desktop Full (권장 - Jetson 성능 충분)
sudo apt install -y ros-humble-desktop

# Option B: ROS Base (저사양 환경)
# sudo apt install -y ros-humble-ros-base

# 필수 도구
sudo apt install -y \
  python3-argcomplete \
  python3-colcon-common-extensions \
  ros-dev-tools
```

> **설치 용량:** 약 1.5~2GB (Desktop 기준)

---

## 2.5 환경 변수 설정

```bash
echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
echo 'export ROS_DOMAIN_ID=30 #TURTLEBOT3' >> ~/.bashrc
source ~/.bashrc
```

---

## 2.6 설치 확인

```bash
# ROS2 환경 변수
printenv | grep ROS

# 패키지 수
ros2 pkg list | wc -l

# 데모 통신 테스트
ros2 run demo_nodes_cpp talker &

# 다른 터미널에서
ros2 run demo_nodes_py listener
```

---

## 2.7 추가 ROS2 패키지 설치

```bash
# 유용한 패키지들 (Jetson에서 네이티브 실행 가능)
sudo apt install -y \
  ros-humble-rqt* \
  ros-humble-turtlesim \
  ros-humble-gazebo-* \
  ros-humble-cartographer \
  ros-humble-cartographer-ros \
  ros-humble-slam-toolbox \
  ros-humble-nav2-* \
  ros-humble-turtlebot3*
```

---

## 2.8 CUDA + ROS2 연동 확인

```bash
# CUDA 라이브러리가 ROS2에서 로드 가능한지 확인
python3 -c "
import ctypes
try:
    ctypes.CDLL('libcudart.so')
    print('✅ CUDA Runtime found')
except:
    print('❌ CUDA Runtime not found')
"
```

---

## 📝 연습 문제

> RPi 버전 Day 2의 연습 문제와 동일합니다.
> 추가: `jtop`으로 ROS2 노드 실행 전후의 GPU 메모리 사용량 차이를 측정하세요.
