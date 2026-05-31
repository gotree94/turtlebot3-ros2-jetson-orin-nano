# Day 8 — TurtleBot3 패키지 설치 (Jetson Orin Nano)

> **목표:** Jetson Orin Nano에 TurtleBot3 구동에 필요한 ROS2 패키지를 설치한다.

---

## 8.1 개요

> **RPi 버전 Day 8과 설치 과정이 거의 동일합니다.**  
> Jetson의 차별점은 **빠른 빌드 시간**과 **Isaac ROS 추가 설치 가능**입니다.

---

## 8.2 필수 시스템 패키지

```bash
sudo apt install -y \
  python3-argcomplete \
  python3-colcon-common-extensions \
  libboost-system-dev \
  build-essential \
  libudev-dev
```

---

## 8.3 TurtleBot3 패키지 설치

### 방법 A: Debian 바이너리 (권장)

```bash
sudo apt install -y \
  ros-humble-hls-lfcd-lds-driver \
  ros-humble-turtlebot3-msgs \
  ros-humble-dynamixel-sdk \
  ros-humble-xacro \
  ros-humble-turtlebot3
```

### 방법 B: 소스 빌드

Jetson Orin Nano에서는 RPi와 달리 **매우 빠른 빌드**가 가능합니다 (약 5분):

```bash
mkdir -p ~/turtlebot3_ws/src
cd ~/turtlebot3_ws/src

git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b humble https://github.com/ROBOTIS-GIT/ld08_driver.git

cd ~/turtlebot3_ws
source /opt/ros/humble/setup.bash
colcon build --symlink-install

echo 'source ~/turtlebot3_ws/install/setup.bash' >> ~/.bashrc
source ~/.bashrc
```

> **빌드 시간 비교:** RPi 4B ~50분 vs Jetson Orin Nano ~5분 (NVMe SSD + 빠른 CPU)

---

## 8.4 OpenCR udev 규칙

```bash
sudo cp $(ros2 pkg prefix turtlebot3_bringup)/share/turtlebot3_bringup/script/99-turtlebot3-cdc.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
sudo usermod -aG dialout $USER
# 재로그인 필요
```

---

## 8.5 OpenCR 펌웨어 확인

```bash
export OPENCR_PORT=/dev/ttyACM0
export OPENCR_MODEL=burger
ls -la /dev/ttyACM0

# 펌웨어 업데이트 (필요 시)
cd ~
wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS2/latest/opencr_firmware.tar.gz
tar xvf opencr_firmware.tar.gz
cd opencr_firmware
./update.sh $OPENCR_PORT $OPENCR_MODEL
```

---

## 8.6 환경 변수

```bash
echo 'export TURTLEBOT3_MODEL=burger' >> ~/.bashrc
echo 'export ROS_DOMAIN_ID=30' >> ~/.bashrc
echo 'export LDS_MODEL=LDS-01' >> ~/.bashrc
echo 'export OPENCR_PORT=/dev/ttyACM0' >> ~/.bashrc
source ~/.bashrc
```

---

## 8.7 (선택) Isaac ROS 개요

NVIDIA Isaac ROS는 Jetson 플랫폼에서 ROS2를 가속하는 라이브러리 모음입니다.
TurtleBot3와 직접 연관은 없지만, 확장을 위해 알아둡니다.

```bash
# Isaac ROS는 NPK (NVIDIA Package Manager)로 설치
# https://developer.nvidia.com/isaac-ros-download

# 주요 패키지:
# - isaac_ros_nitros: GPU 가속 ROS2 메시지
# - isaac_ros_dnn_inference: DNN 추론 파이프라인
# - isaac_ros_visual_slam: GPU 가속 Visual SLAM
```

---

## 8.8 빌드 속도 확인

```bash
# 빌드 시간 테스트
time colcon build --symlink-install
```

---

## 📝 연습 문제

> RPi 버전 Day 8과 동일
> 추가: 빌드 시간을 `time` 명령어로 측정하고 결과를 기록하세요
