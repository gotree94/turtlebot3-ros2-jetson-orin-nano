# Week 1 — 환경 구축 & ROS2 기초 (Jetson Orin Nano)

> **목표:** Jetson Orin Nano에 JetPack 6 + ROS2 Humble을 설치하고 ROS2 개념을 마스터한다.

---

## 📅 주간 일정

| 일차 | 주제 | 핵심 내용 |
|------|------|----------|
| Day 1 | Jetson OS 설치 | JetPack 6.x 설치 (SDK Manager 또는 직접 플래싱) |
| Day 2 | ROS2 Humble 설치 | ROS2 Base 설치 (Ubuntu 22.04 aarch64) |
| Day 3 | Remote PC 세팅 (선택) | 필요 시 PC에 ROS2 Desktop 설치 |
| Day 4 | ROS2 핵심 개념 | Nodes, Topics, Services, Actions |
| Day 5 | 첫 ROS2 패키지 | Publisher/Subscriber 작성 |
| Day 6 | 복습 & GPU 설정 | CUDA 확인, nvtop, GPU 모니터링 |
| Day 7 | 미니 프로젝트 | GPU/CPU 온도 모니터링 |

---

## ⚠️ RPi 버전과의 주요 차이점

| 항목 | RPi 4B | Jetson Orin Nano |
|------|--------|-------------------|
| OS 설치 | Raspberry Pi Imager | SDK Manager / 직접 플래싱 |
| 스토리지 | microSD | **NVMe SSD 권장** |
| GPU | 없음 | CUDA 1024 cores |
| SLAM/Nav2 | Remote PC 필요 | **온보드 실행 가능** |
| 전원 | USB-C 5V 3A | USB-C PD 15V~45W |
| ROS2 Humble 설치 | apt (동일) | apt (동일, aarch64) |

> 💡 **Day 3 (Remote PC)는 선택 사항입니다.** Jetson Orin Nano는 충분한 성능으로 RViz2, rqt, SLAM, Nav2를 모두 자체 실행할 수 있습니다. 단, 모니터/키보드가 연결되어 있거나 VNC/XRDP로 원격 데스크톱 접속이 필요합니다.
