# 문제 해결 가이드 (Jetson Orin Nano)

> **RPi 버전의 일반 문제 해결:** `../turtlebot3-ros2-curriculum/appendices/troubleshooting.md` 참조
>
> 아래는 Jetson Orin Nano 특화 문제입니다.

---

## 1. JetPack / SDK Manager 문제

### SDK Manager 실행 안 됨

```bash
# Docker 권한 문제
sudo usermod -aG docker $USER
newgrp docker

# 재시도
./sdkmanager
```

### JetPack 설치 실패 (네트워크)

```bash
# Host PC에서 수동 다운로드
# NVIDIA SDK Manager → Download for target → Save .deb/.tar

# Jetson에서 오프라인 설치
sudo apt install ./<deb_file>
```

---

## 2. CUDA / GPU 문제

### CUDA 미인식

```bash
# CUDA 경로 확인
ls /usr/local/cuda/bin/nvcc
nvcc --version

# PATH 누락 시
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

# .bashrc에 추가
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
```

### GPU 메모리 부족

```bash
# GPU 메모리 상태 확인
sudo tegrastats

# 해결: 불필요한 GPU 프로세스 종료
sudo systemctl stop nvargus-daemon  # 카메라 데몬 중지

# YOLO 모델 축소
python3 -c "from ultralytics import YOLO; YOLO('yolov8n.pt')"  # nano 사용
```

### torch.cuda.is_available() = False

```bash
# PyTorch가 Jetson용인지 확인
python3 -c "import torch; print(torch.__version__)"

# NVIDIA 공식 PyTorch 설치 (pip 아님!)
# https://forums.developer.nvidia.com/t/pytorch-for-jetson/
pip3 install --no-cache https://developer.download.nvidia.com/.../torch-2.1.0-cp310-cp310-linux_aarch64.whl
```

---

## 3. 전원 / 온도 문제

### 쓰로틀링 발생

```bash
# 현재 전원 모드 확인
sudo nvpmodel -q

# MAXN 모드로 전환 (최대 성능)
sudo nvpmodel -m 0

# 쿨링팬 수동 제어
sudo sh -c 'echo 255 > /sys/devices/pwm-fan/target_pwm'  # 100%
```

### 과열

```bash
# 온도 확인
sudo tegrastats | grep "GPU@"

# CPU/GPU 온도 모니터링
watch -n 1 'cat /sys/class/thermal/thermal_zone*/temp'

# 권장: 70°C 이상이면 열관리 필요
# 방열판 접촉 확인, 쿨링팬 작동 확인
```

---

## 4. ROS2 관련 문제

### ROS2 Humble 패키지 누락

```bash
# Ubuntu aarch64 확인
dpkg --print-architecture  # arm64여야 함

# ROS2 저장소 추가
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update

# 다시 설치
sudo apt install ros-humble-desktop
```

### colcon build arm64 이슈

```bash
# aarch64 네이티브 빌드 (에뮬레이션 아님)
uname -m  # aarch64 확인
# → 문제없음. Jetson은 네이티브 arm64
```

### RViz2 실행 안 됨

```bash
# GPU 드라이버 문제일 가능성
sudo apt install --reinstall nvidia-driver-*

# NVIDIA X Server 설정
sudo nvidia-xconfig

# 재부팅 후 확인
reboot
```

---

## 5. YOLO / AI 문제

### ultralytics 설치 실패

```bash
# aarch64 호환 확인
pip3 install ultralytics --no-deps
pip3 install -r requirements.txt  # 수동 의존성 설치
```

### 카메라 노드 미작동

```bash
# USB 카메라 확인
ls /dev/video*
v4l2-ctl --list-devices

# 권한 문제
sudo usermod -aG video $USER
newgrp video
```

### YOLO 추론 속도 느림

```bash
# CPU에서 실행 중인지 확인
python3 -c "
from ultralytics import YOLO
model = YOLO('yolov8n.pt')
print('Device:', model.device)  # cuda:0가 아니면 GPU 미사용
"

# 모델을 GPU로 명시적 이동
model.to('cuda')
```

---

## 6. 저장 공간 문제

### eMMC 부족 (NVMe 미사용 시)

```bash
# 디스크 사용량 확인
df -h

# 불필요 파일 정리
sudo apt autoremove
sudo apt autoclean
sudo journalctl --vacuum-size=500M

# Docker 이미지 정리
docker system prune -a
```

---

## 7. 네트워크 문제

### WiFi 연결 불안정

```bash
# NetworkManager 재시작
sudo systemctl restart NetworkManager

# 5GHz 대역 사용
nmcli device wifi list
nmcli device wifi connect <SSID> password <PW>

# WiFi 전원 관리 비활성화
iwconfig wlan0 power off
```

### ROS_DOMAIN_ID 미설정

```bash
echo 'export ROS_DOMAIN_ID=30' >> ~/.bashrc
source ~/.bashrc
```

---

## 문제 해결이 안 될 때

1. **NVIDIA Jetson Developer Forums**: https://forums.developer.nvidia.com/c/agx-autonomous-machines/jetson-embedded-systems/
2. **Jetson Zoo**: https://elinux.org/Jetson_Zoo (추가 패키지)
3. **NVIDIA Developer Docs**: https://docs.nvidia.com/jetson/
