# 참고 자료 (Jetson Orin Nano)

> **RPi 버전 참고 자료:** `../turtlebot3-ros2-curriculum/appendices/references.md` 참조
>
> 아래는 Jetson Orin Nano 및 GPU 가속 로보틱스 관련 추가 자료입니다.

---

## 공식 문서

| 자료 | 설명 | 링크 |
|------|------|------|
| NVIDIA Jetson Orin Nano | 제품 페이지 | https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/ |
| JetPack SDK | SDK 문서 | https://developer.nvidia.com/embedded/jetpack |
| SDK Manager | 설치 도구 | https://developer.nvidia.com/sdk-manager |
| NVIDIA Developer | Jetson 포럼 | https://forums.developer.nvidia.com/c/agx-autonomous-machines/jetson-embedded-systems/ |
| Jetson Zoo | 추가 패키지 | https://elinux.org/Jetson_Zoo |

## GPU 가속 라이브러리

| 라이브러리 | 설명 | 설치 |
|-----------|------|------|
| CUDA 12.x | GPU 병렬 컴퓨팅 | JetPack 포함 |
| cuDNN 8.x | 딥러닝 최적화 | JetPack 포함 |
| TensorRT | 추론 최적화 | JetPack 포함 |
| PyTorch for Jetson | NVIDIA 제공 빌드 | `pip3 install torch...` |
| ultralytics | YOLOv8 | `pip3 install ultralytics` |
| CuPy | GPU 배열 연산 | `pip3 install cupy-cuda12x` |
| RAPIDS | GPU 데이터 사이언스 | `pip3 install cudf-cu12` |

## 관련 프로젝트

| 프로젝트 | 설명 | 링크 |
|---------|------|------|
| Isaac ROS | GPU 가속 ROS2 패키지 | https://developer.nvidia.com/isaac-ros-gems/ |
| Isaac Sim | 로봇 시뮬레이션 | https://developer.nvidia.com/isaac-sim |
| NVIDIA AI Center | 사전 학습 모델 | https://catalog.ngc.nvidia.com/ |
| JetBot | NVIDIA 오픈소스 로봇 | https://github.com/NVIDIA-AI-IOT/jetbot |
| ros2_jetson_stats | Jetson 정보 ROS2 패키지 | https://github.com/rbonghi/jetson_stats |

## 교육 자료

| 자료 | 설명 |
|------|------|
| NVIDIA DLI | https://www.nvidia.com/dli/ — Jetson 및 딥러닝 강좌 |
| Jetson AI Courses | https://developer.nvidia.com/embedded/learn/jetson-ai-certification |
| ROS2 + Isaac ROS | NVIDIA 공식 튜토리얼 |
| Hello AI World | https://github.com/dusty-nv/jetson-inference |

## 성능 참고

| 항목 | Jetson Orin Nano 8GB |
|------|---------------------|
| GPU | 1024-core Ampere |
| Tensor Cores | 32 |
| GPU 성능 | 40 TOPS (INT8) |
| CPU | 6-core Cortex-A78AE v8.2 64-bit |
| 메모리 | 8GB 128-bit LPDDR5 |
| 전력 | 7W ~ 15W |
| ROS2 빌드 | ~5분 (전체 workspace) |
| YOLOv8n | ~60 FPS |
| SLAM + Nav2 + RViz2 | 동시 가능 |
