# 🐢 TurtleBot3 Burger + Jetson Orin Nano + ROS2 Humble

> **Jetson Orin Nano** 환경에서 ROS2 Humble을 구축하고, TurtleBot3 Burger를 제어하며 SLAM/네비게이션/Edge AI까지 마스터하는 3주 집중 코스

---

## 📋 개요

이 커리큘럼은 **TurtleBot3 Burger** 로봇에 **NVIDIA Jetson Orin Nano (8GB/16GB)**를 SBC로 탑재하고,<br> **ROS2 Humble Hawksbill** 환경에서 최종 프로젝트까지 완성하는 것을 목표로 합니다.

| 항목 | 내용 |
|------|------|
| **총 기간** | 3주 (21일) |
| **난이도** | 초중급 (Linux 기초 지식 필요) |
| **ROS 버전** | ROS2 Humble Hawksbill |
| **타겟 보드** | NVIDIA Jetson Orin Nano (8GB/16GB) |
| **OS** | Ubuntu 22.04 LTS aarch64 (JetPack 6.x) |
| **로봇** | TurtleBot3 Burger |
| **Remote PC** | 선택 사항 (Jetson 자체로 GUI+SLAM+Nav2 동시 수행 가능) |

---

## 🛠 사전 준비물

### 하드웨어

| 항목 | 비고 |
|------|------|
| TurtleBot3 Burger | OpenCR 1.0, DYNAMIXEL XL430-W250 x2, 배터리 포함 |
| NVIDIA Jetson Orin Nano | 8GB 또는 16GB (Developer Kit 권장) |
| NVMe SSD | 128GB 이상 (M.2 Key M) — **필수** (eMMC만으로는 부족) |
| MicroSD 카드 | JetPack 플래싱용 (선택) |
| USB-C 전원 어댑터 | Jetson Orin Nano 공식 어댑터 (USB-C PD, 15V/3A~45W) |
| WiFi 모듈 | Intel AX210 또는 호환 M.2 Key E 모듈 (선택, 유선 이더넷 권장) |
| Remote PC (노트북/데스크탑, 선택) | Ubuntu 22.04 Desktop — 원격 GUI용 |
| WiFi 공유기 | 동일 네트워크 |
| USB 카메라 (선택) | TurtleBot3 Burger 마운트 가능한 모델 |
| 방열판 + 쿨링팬 | Jetson Orin Nano 발열 대책 — **필수** |

### 소프트웨어

| 항목 | 버전 |
|------|------|
| NVIDIA JetPack | 6.0 이상 (Ubuntu 22.04 aarch64) |
| SDK Manager | 최신 버전 (호스트 PC 필요) |
| CUDA | 12.x (JetPack 포함) |
| ROS 2 | Humble Hawksbill |
| Gazebo | Classic 11 (시뮬레이션) |

---

## Jetson Orin Nano vs Raspberry Pi 4B 비교

| 항목 | Jetson Orin Nano (8GB) | Raspberry Pi 4B (8GB) |
|------|------------------------|----------------------|
| **CPU** | ARM Cortex-A78AE x6 @1.5GHz | ARM Cortex-A72 x4 @1.8GHz |
| **GPU** | NVIDIA Ampere 1024 CUDA cores | ❌ 없음 |
| **AI 성능** | 40 TOPS (INT8) | ❌ 불가 |
| **메모리 대역폭** | 68 GB/s | ~3 GB/s |
| **스토리지** | NVMe SSD (권장) | microSD |
| **전력** | 7~15W | 5~7W |
| **SLAM+Nav2** | ✅ 완전 온보드 실행 가능 | ❌ Remote PC 필요 |
| **객체 탐지** | ✅ 실시간 YOLO 가능 | ❌ 불가 |

---

## 📚 주차별 학습 로드맵

```
Week 1 ────┬── Day 1~2: JetPack OS + ROS2 설치 (Jetson)
           ├── Day 3~4: Remote PC + ROS2 개념 (RPi와 동일)
           ├── Day 5~6: 첫 패키지 + 보너스: CUDA 기초
           └── Day 7: [미니 프로젝트] GPU 온도 모니터링
                   
Week 2 ────┬── Day 8~9: TurtleBot3 SBC + PC 패키지
           ├── Day 10~11: Bringup + GPU 가속 센서 시각화
           ├── Day 12~13: PID 제어 + 보너스: Isaac ROS 개요
           └── Day 14: [미니 프로젝트] LIDAR 장애물 회피

Week 3 ────┬── Day 15~16: SLAM 이론 + 온보드 매핑
           ├── Day 17~18: Navigation2 + 온보드 자율주행
           └── Day 19~21: [최종 프로젝트] AI 기반 자율 순찰 로봇
```

---

## 🔑 이 버전의 핵심 차별점

이 커리큘럼은 Raspberry Pi 4B 버전과 동일한 3주 구조를 따르지만, **Jetson Orin Nano**의 강력한 GPU와 AI 가속 기능을 활용합니다:

1. **온보드 SLAM + Nav2**: Remote PC 없이 Jetson 단독으로 SLAM과 자율주행 실행
2. **GPU 가속 시각화**: RViz2, rqt 등을 Jetson 디스플레이에서 직접 실행
3. **AI/ML 통합**: 최종 프로젝트에서 실시간 객체 탐지(YOLO) + 순찰 시스템 결합
4. **CUDA 활용**: 센서 데이터 전처리 GPU 가속
5. **Isaac ROS**: NVIDIA의 ROS2 기반 로봇 가속 라이브러리 활용 가능

---

## 📂 디렉토리 구조

Jetson 버전은 Raspberry Pi 버전과 **동일한 디렉토리 구조**를 유지하며, 내용이 동일한 파일은 상호 참조합니다.

```
turtlebot3-ros2-curriculum-jetson/
├── README.md
├── week1/
│   ├── README.md
│   ├── day01_jetson_setup.md         ← Jetson 전용 (JetPack 설치)
│   ├── day02_ros2_installation.md     ← RPi와 동일
│   ├── day03_remote_pc_setup.md       ← RPi와 동일
│   ├── day04_ros2_concepts.md         ← RPi와 동일
│   ├── day05_first_package.md         ← RPi와 동일
│   ├── day06_review_gpu_setup.md      ← GPU 설정 포함 변형
│   └── day07_mini_project_gpu_temp.md ← GPU 온도 모니터링
├── week2/
│   ├── README.md
│   ├── day08_turtlebot3_sbc_setup.md  ← Jetson 패키지 설치
│   ├── day09_remote_pc_tb3_pkg.md     ← RPi와 동일
│   ├── day10_bringup_teleop.md        ← RPi와 동일
│   ├── day11_sensor_visualization.md  ← Jetson 네이티브 실행
│   ├── day12_pid_control_basics.md    ← RPi와 동일
│   ├── day13_rviz2_urdf.md           ← RPi와 동일
│   └── day14_mini_project_obstacle.md ← RPi와 동일
├── week3/
│   ├── README.md
│   ├── day15_slam_theory.md           ← RPi와 동일
│   ├── day16_slam_mapping_jetson.md   ← 온보드 SLAM
│   ├── day17_navigation2_concepts.md  ← RPi와 동일
│   ├── day18_nav2_autonomous_jetson.md← 온보드 Nav2
│   ├── day19_final_project_plan.md    ← AI 순찰 기획
│   ├── day20_final_project_impl_ai.md ← AI 객체 탐지 통합
│   └── day21_final_project_final.md   ← 최종 완성
├── exercises/
│   └── ...                            ← RPi와 동일 (보충 문제 추가)
└── appendices/
    ├── troubleshooting.md
    ├── ros2_cheatsheet.md
    └── references.md
```

> 💡 **RPi 버전과 내용이 동일한 파일**은 `RPi 버전과 동일`으로 표기하고, Jetson 전용 내용만 차별화하여 작성합니다.

---

## 🚀 빠른 시작

```bash
git clone https://github.com/your-username/turtlebot3-ros2-curriculum-jetson.git
cd turtlebot3-ros2-curriculum-jetson
# week1/day01부터 순서대로 진행
```

---

## 📝 라이선스

MIT License
