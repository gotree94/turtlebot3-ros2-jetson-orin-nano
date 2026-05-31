# Day 6 — 복습 & GPU 설정

> **목표:** Week 1을 복습하고, Jetson Orin Nano의 GPU를 ROS2에서 활용할 준비를 한다.

---

## 6.1 Week 1 핵심 요약

> ROS2 명령어 요약은 **RPi 버전 Day 6**와 동일합니다.
> 아래는 Jetson 추가 내용입니다.

---

## 6.2 CUDA 개발 환경 확인

```bash
# CUDA 설치 확인
nvcc --version

# CUDA 샘플 테스트 (선택)
cd /usr/local/cuda/samples/1_Utilities/deviceQuery
sudo make
./deviceQuery

# 출력에 "Detected 1 CUDA Capable device" 확인
# Name: "NVIDIA Tegra X1" 또는 "NVIDIA Orin"
```

### cuDNN 확인

```bash
# cuDNN 버전 확인
dpkg -l | grep cudnn

# 또는
cat /usr/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

### TensorRT 확인

```bash
# TensorRT 버전
dpkg -l | grep tensorrt

# Python에서 확인
python3 -c "import tensorrt; print(tensorrt.__version__)"
```

---

## 6.3 Jetson Stats 모니터링

```bash
# jtop 설치 (jetson_stats 패키지)
sudo pip3 install jetson-stats
sudo jtop
```

`jtop`에서 확인 가능한 정보:
- **CPU**: 코어별 사용률, 주파수
- **GPU**: 사용률, 주파수, 메모리
- **RAM**: 시스템 메모리 사용량
- **SWAP**: 스왑 메모리
- **NVMe**: 디스크 I/O
- **온도**: CPU/GPU/보드 온도
- **전력**: 현재 전력 소비(W)
- **전원 모드**: 현재 nvpmodel

---

## 6.4 ROS2 + GPU 연동 가능성

Jetson Orin Nano에서 ROS2와 GPU를 연동하는 방법:

### 1) Vision 관련

```bash
# OpenCV CUDA 지원 확인
python3 -c "
import cv2
print('CUDA Enabled:', cv2.cuda.getCudaEnabledDeviceCount())
"
```

### 2) Isaac ROS (NVIDIA 로봇 가속 라이브러리)

```bash
# Isaac ROS는 별도 설치 필요
# https://developer.nvidia.com/isaac-ros-download
```

### 3) Point Cloud 처리

LIDAR 포인트 클라우드 처리를 GPU로 가속 가능:
- `PCL` (Point Cloud Library) — CUDA 지원
- Custom CUDA kernels로 scan 필터링

---

## 6.5 성능 최적화 팁

### 전원 모드별 성능

```bash
# MAXN 모드 (최대 성능)
sudo nvpmodel -m MAXN
sudo jetson_clocks  # 최대 클럭 고정

# 15W 모드 (전력 효율)
sudo nvpmodel -m 0
```

### 메모리 최적화

```bash
# ZRAM 설정 (추가 메모리 압축)
sudo apt install -y zram-config

# 불필요한 서비스 중지
sudo systemctl disable --now snapd  # Snap 불필요 시
sudo systemctl disable --now bluetooth  # 블루투스 불필요 시
```

### Docker + ROS2

```bash
# NVIDIA Container Toolkit
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker

# ROS2 Humble Docker 이미지 (선택)
docker pull ros:humble-ros-base-jammy
docker run -it --rm --runtime nvidia \
  -e NVIDIA_VISIBLE_DEVICES=all \
  ros:humble-ros-base-jammy
```

---

## 📝 연습 문제

1. `deviceQuery` CUDA 샘플을 실행하고, GPU의 Compute Capability 버전을 기록하세요
2. `jtop`을 실행하고 CPU/GPU 온도, 메모리 사용량을 스크린샷으로 저장하세요
3. `sudo jetson_clocks` 실행 후 `jtop`으로 클럭 변화를 확인하세요
4. CUDA 툴킷 버전과 TensorRT 버전을 확인하고 호환성을 문서화하세요
5. ROS2 talker/listener 실행 중 GPU 메모리 사용량 변화를 `jtop`으로 모니터링하세요
