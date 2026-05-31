# Day 1 — Jetson Orin Nano OS 설치 & 초기 설정

> **목표:** Jetson Orin Nano에 JetPack 6.x (Ubuntu 22.04 aarch64)를 설치하고 SSH 원격 접속까지 확인한다.

---

## 1.1 개요

NVIDIA Jetson Orin Nano는 TurtleBot3 Burger의 SBC로 탁월한 선택입니다.
Raspberry Pi 4B 대비 **10배 이상의 GPU 성능**과 **20배 이상의 메모리 대역폭**을 제공하여, SLAM/Nav2/AI 추론을 모두 온보드에서 처리할 수 있습니다.

> **JetPack이란?** NVIDIA Jetson 보드용 공식 SDK로, Ubuntu OS + CUDA + cuDNN + TensorRT + Multimedia API를 포함하는 올인원 패키지입니다.

---

## 1.2 준비물

- NVIDIA Jetson Orin Nano Developer Kit (8GB 또는 16GB)
- NVMe SSD (M.2 Key M, 256GB 이상 권장) — **사전에 보드에 장착**
- MicroSD 카드 (32GB+, 초기 플래싱용)
- USB-C 전원 어댑터 (Jetson Orin Nano 정품, 15V/3A~45W)
- **호스트 PC** (x86_64, Ubuntu 20.04/22.04 또는 Windows, SDK Manager 설치용)
- USB 케이블 (Jetson Micro-USB → 호스트 PC, 리커버리 모드 진입용)
- HDMI 모니터 + USB 키보드/마우스 (초기 설정용)
- (선택) WiFi 모듈 (Intel AX210 등 M.2 Key E)
- (선택) 이더넷 케이블

---

## 1.3 NVMe SSD 장착 (사전 작업)

Jetson Orin Nano Developer Kit에는 NVMe SSD 슬롯이 있습니다.

1. 보드 하단의 M.2 Key M 슬롯 확인
2. NVMe SSD (2280 규격)를 45도 각도로 삽입
3. 나사로 고정
4. 방열판 부착 (SSD 발열 관리)

> ⚠️ **NVMe SSD는 선택이 아닌 사실상 필수입니다.** eMMC(기본 8GB)만으로는 ROS2 + TurtleBot3 + AI 패키지를 설치하기에 절대적으로 부족합니다.

---

## 1.4 JetPack 6 설치 방법

### 방법 A: SDK Manager 사용 (권장, 간편)

**호스트 PC (x86_64) 필요**

#### Step 1: SDK Manager 설치 (호스트 PC)

```bash
# NVIDIA SDK Manager 다운로드
# https://developer.nvidia.com/sdk-manager 에서 .deb 파일 다운로드

# 설치 (Ubuntu)
sudo apt install ./sdkmanager_[version].deb

# 실행
sdkmanager
```

#### Step 2: Jetson 준비

1. **Jetson Orin Nano를 리커버리 모드로 부팅:**
   - 전원 연결 **해제**
   - Micro-USB 케이블로 Jetson ←→ 호스트 PC 연결
   - **리커버리 버튼(FORCE RECOVERY)** 을 누른 상태로 유지
   - **리셋 버튼(RESET)** 을 1초 누름
   - 리커버리 버튼에서 손을 뗌
   - 호스트 PC에서 `lsusb` 명령어로 `NVidia Corp. APX` 장치 확인

#### Step 3: SDK Manager에서 설치

1. SDK Manager 실행 → "Jetson Orin Nano" 선택
2. JetPack 6.0 이상 선택
3. **Storage Device**: NVMe SSD 선택 (중요! eMMC 선택 시 용량 부족)
4. 구성 요소 선택:
   - ✅ Jetson Linux (필수)
   - ✅ CUDA 12.x (필수)
   - ✅ cuDNN (권장)
   - ✅ TensorRT (권장)
   - ✅ OpenCV for Jetson (권장)
   - ✅ VPI (Vision Programming Interface) (선택)
5. **Install** 클릭 → 약 30분~1시간 소요
6. 설치 완료 후 Jetson 자동 재부팅

### 방법 B: 직접 이미지 플래싱 (호스트 PC 없이)

Jetson Orin Nano Developer Kit 전용으로 사전 빌드된 SD 카드 이미지를 사용:

1. [NVIDIA Embedded Download Center](https://developer.nvidia.com/embedded/downloads)에서
   **Jetson Orin Nano SD Card Image** 다운로드
2. balenaEtcher 또는 dd로 MicroSD에 이미지 쓰기:
   ```bash
   # Linux
   sudo dd if=jetson-orin-nano-sd-image.img of=/dev/sdX bs=1M status=progress
   ```
3. MicroSD 삽입 후 부팅
4. **주의:** SD 카드 부팅은 NVMe보다 느리므로, 부팅 후 시스템을 NVMe로 복사 권장

---

## 1.5 초기 부팅 & 설정

### Step 1: 전원 연결 및 부팅

1. 모니터, 키보드, 마우스 연결
2. USB-C 전원 연결
3. 초기 부팅 시 NVIDIA 로고 → Ubuntu OOBE (Out-Of-Box Experience) 진행

### Step 2: Ubuntu 초기 설정

```
1. 언어: English (US)
2. 키보드 레이아웃: Korean 또는 English (US)
3. 사용자 계정 생성:
   Username: jetson (또는 ubuntu)
   Password: [강력한 비밀번호]
4. Hostname: jetson-orin (또는 turtlebot-jetson)
5. NVIDIA 계정 로그인 (선택, 건너뛰기 가능)
6. 파티션 설정: 전체 NVMe 사용 (권장)
```

### Step 3: 시스템 업데이트

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
sudo reboot
```

### Step 4: 기본 패키지 설치

```bash
# 필수 도구
sudo apt install -y \
  build-essential \
  cmake \
  git \
  curl \
  wget \
  vim \
  nano \
  htop \
  tree \
  net-tools \
  openssh-server

# 시스템 모니터링
sudo apt install -y \
  nvtop \
  jtop \
  python3-pip \
  python3-venv

# SSH 활성화
sudo systemctl enable --now sshd
```

---

## 1.6 CUDA & GPU 확인

### CUDA 버전 확인

```bash
# CUDA 설치 확인
nvcc --version

# 출력 예시:
# nvcc: NVIDIA (R) Cuda compiler driver
# Copyright (c) 2005-2024 NVIDIA Corporation
# Built on ...
# Cuda compilation tools, release 12.x, V12.x.xx
```

### GPU 상태 확인

```bash
# GPU 실시간 모니터링
sudo nvtop

# 또는
jtop  # jetson_stats 패키지
```

### GPU 정보 상세 확인

```bash
# GPU 정보
cat /proc/device-tree/model
# NVIDIA Jetson Orin Nano Developer Kit

# GPU 아키텍처
nvidia-smi  # (Jetson에서는 제한적 지원)
# 대신 tegrastats 사용
sudo tegrastats
```

---

## 1.7 네트워크 설정

### WiFi 설정 (M.2 WiFi 모듈 장착 시)

```bash
# WiFi 장치 확인
iwconfig
ip addr show

# NetworkManager로 WiFi 연결
nmcli dev wifi list
sudo nmcli dev wifi connect "WiFiSSID" password "WiFiPassword"

# 또는 데스크톱 GUI에서 WiFi 설정
```

### 고정 IP 설정 (선택)

```bash
nmcli connection show
sudo nmcli connection modify "Wired connection 1" \
  ipv4.addresses 192.168.x.x/24 \
  ipv4.gateway 192.168.x.1 \
  ipv4.dns 8.8.8.8 \
  ipv4.method manual
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

### SSH 접속 확인 (같은 네트워크 PC에서)

```bash
# 호스트 PC에서
ssh jetson@jetson-orin.local
# 또는
ssh jetson@192.168.x.x
```

---

## 1.8 NVMe → 추가 공간 확인

```bash
# 디스크 확인
lsblk

# 예시 출력:
# nvme0n1     259:0    0 232.9G  0 disk
# ├─nvme0n1p1  ...     0   512M  0 part /boot/efi
# └─nvme0n1p2  ...     0 232.4G  0 part /

# 사용 가능 공간
df -h
```

---

## 1.9 전원 모드 설정 (성능 최적화)

Jetson Orin Nano는 다양한 전원 모드를 지원합니다:

```bash
# 현재 전원 모드 확인
sudo nvpmodel -q

# 최대 성능 모드 (15W, GPU 클럭 최대)
sudo nvpmodel -m 0

# MAXN 모드 (추가 오버클럭, 발열 관리 필요)
sudo nvpmodel -m MAXN

# 전원 모드 목록
sudo nvpmodel -p --verbose

# 부팅 시 자동 적용
sudo nvpmodel -m 0
```

---

## 1.10 확인 사항

```bash
# 호스트네임 확인
hostname

# IP 주소 확인
hostname -I

# 시스템 정보
uname -a

# Jetson 모델 확인
cat /proc/device-tree/model

# 메모리
free -h

# 디스크
df -h

# 네트워크 (다른 PC와 ping)
ping 192.168.x.x

# GPU 온도
cat /sys/devices/virtual/thermal/thermal_zone*/temp
```

---

## 📝 연습 문제

1. `jtop` 명령어를 설치하고 실행하여 Jetson Orin Nano의 CPU/GPU/메모리 사용률을 실시간으로 모니터링하세요
2. `sudo tegrastats`로 GPU 코어 주파수와 메모리 사용량을 확인하세요
3. CUDA 버전(`nvcc --version`)과 cuDNN 버전을 확인하세요 (`dpkg -l | grep cudnn`)
4. 전원 모드를 MAXN과 15W 모드로 변경하면서 성능 차이를 `jtop`으로 측정하세요
5. `nvtop`을 설치하고 GPU 프로세스 리스트를 확인하세요

---

## ⚠️ 트러블슈팅

| 문제 | 해결 방법 |
|------|----------|
| SDK Manager가 Jetson을 찾지 못함 | 리커버리 모드 재확인, Micro-USB 케이블 교체 |
| 부팅 후 NVMe 미인식 | NVMe SSD 재장착, 포맷 필요 시 `sudo fdisk /dev/nvme0n1` |
| CUDA가 설치 안 됨 | `sudo apt install nvidia-cuda-toolkit` 또는 SDK Manager 재실행 |
| WiFi 모듈 미인식 | M.2 Key E 슬롯 연결 확인, 호환 모델(Intel AX210) 사용 |
| 과열로 시스템 종료 | 전원 모드를 낮추거나(15W) 쿨링팬 연결 확인 |
| SD 카드 부팅만 가능 | NVMe로 시스템 복사 (부트로더 설정 변경 필요) |
