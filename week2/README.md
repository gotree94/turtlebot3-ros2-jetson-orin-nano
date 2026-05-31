# Week 2 — TurtleBot3 구동 & 센서/제어 (Jetson Orin Nano)

> **목표:** 실제 TurtleBot3 Burger를 구동하고, 센서 데이터를 GPU 가속으로 처리하며 제어한다.

---

## 📅 주간 일정

| 일차 | 주제 | RPi 버전 대비 |
|------|------|-------------|
| Day 8 | TurtleBot3 패키지 설치 (Jetson) | 🔶 Jetson 전용 (Isaac ROS, CUDA) |
| Day 9 | Remote PC 패키지 (선택) | ✅ 동일 |
| Day 10 | Bringup & Teleoperation | ✅ 동일 |
| Day 11 | 센서 데이터 & RViz2 | 🔶 Jetson 네이티브 실행 (Remote PC 불필요) |
| Day 12 | PID 제어 & 로봇 구동 | ✅ 동일 |
| Day 13 | RViz2 심화 & URDF | ✅ 동일 |
| Day 14 | 미니 프로젝트: 장애물 회피 | ✅ 동일 |

---

## Jetson Orin Nano의 장점 (Week 2)

1. **RViz2를 Jetson 자체 디스플레이에서 직접 실행** — Remote PC 불필요
2. **SLAM/장애물 회피 노드를 Jetson에서 직접 GPU 가속으로 실행 가능**
3. **NVMe SSD 덕분에 빠른 빌드 시간** (RPi microSD 대비 5~10배 빠름)
4. **CUDA 커널로 LIDAR 데이터 전처리 가능** (선택 사항)

> 💡 **Day 11, 13은 RViz2를 Jetson 모니터에서 직접 띄우는 방식으로 진행합니다.**  
> 원격 접속이 필요한 경우 VNC(X11VNC) 또는 X forwarding(X2Go)을 사용할 수 있습니다.
