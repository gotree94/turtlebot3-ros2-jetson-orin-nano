# ROS2 Humble Cheatsheet (Jetson Orin Nano)

> **RPi 버전과 완전 동일합니다.** 아래는 Jetson에 특화된 명령어만 추가합니다.

---

## Jetson 특화 명령어

### 시스템 정보

```bash
# Jetson 모델 확인
cat /proc/device-tree/model
# → NVIDIA Jetson Orin Nano Developer Kit

# CUDA 버전
nvcc --version
# → Cuda compilation tools, release 12.x

# TensorRT 버전
dpkg -l | grep tensorrt

# 전원 모드
sudo nvpmodel -q

# 실시간 모니터링
jtop
```

### Docker (옵션)

```bash
# NVIDIA Container Toolkit
docker run --rm --runtime nvidia nvidia/cuda:12.2-base nvidia-smi

# ROS2 Humble Docker (공식이미지 부재 시)
docker run -it --rm --runtime nvidia \
  -e ROS_DOMAIN_ID=30 \
  --network host \
  ros:humble-ros-base-jammy
```

---

> **전체 ROS2 명령어:** `../turtlebot3-ros2-curriculum/appendices/ros2_cheatsheet.md` 참조
>
> 포함 내용: nodes, topics, services, actions, bags, tf2, rqt, colcon 완전 동일
