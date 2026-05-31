# Day 11 — 센서 데이터 이해 & RViz2 시각화 (Jetson 네이티브)

> **목표:** Jetson Orin Nano에서 직접 RViz2를 실행하여 TurtleBot3의 센서 데이터를 시각화한다.

---

## 11.1 개요

Jetson Orin Nano의 핵심 장점 중 하나는 **디스플레이가 연결된 경우 Remote PC 없이 RViz2를 직접 실행**할 수 있다는 점입니다.

> **Remote PC가 필요 없습니다.**  
> Jetson에 HDMI 모니터 연결 → RViz2, rqt를 모두 Jetson에서 네이티브 실행합니다.

---

## 11.2 디스플레이 설정

### 모니터 직접 연결 (권장)

```bash
# Jetson HDMI 포트에 모니터 연결
# USB 키보드/마우스 연결

# 디스플레이 해상도 확인
xrandr

# 필요 시 해상도 변경
xrandr --output HDMI-0 --mode 1920x1080
```

### 헤드리스 운영 시 (모니터 없음)

VNC 서버 설정:

```bash
# x11vnc 설치
sudo apt install -y x11vnc

# VNC 비밀번호 설정
x11vnc -storepasswd

# 실행
x11vnc -forever -display :0 -usepw

# 클라이언트에서 VNC 접속 (같은 네트워크)
# 주소: jetson-orin.local:5900
```

---

## 11.3 RViz2 네이티브 실행

```bash
# Jetson에서 TurtleBot3 Bringup 중이면
# 같은 Jetson에서 RViz2 직접 실행
rviz2
```

**RViz2 설정 (RPi 버전과 동일):**
1. Fixed Frame → `odom`
2. Add → `RobotModel`
3. Add → `LaserScan` → Topic: `/scan`
4. Add → `Odometry` → Topic: `/odom`
5. Add → `TF`

> **GPU 가속:** Jetson Orin Nano의 GPU가 RViz2의 3D 렌더링을 가속하므로, RPi보다 훨씬 부드러운 시각화가 가능합니다.

---

## 11.4 GPU 가속 확인

```bash
# RViz2 실행 중 GPU 사용률 확인
sudo nvtop

# 또는
jtop
# GPU 섹션에서 RViz2 프로세스 확인
```

---

## 11.5 rqt 네이티브 실행

```bash
# rqt 도구들 모두 Jetson에서 직접 실행 가능
rqt_graph
rqt_console
rqt_plot /scan/ranges[180]
```

---

## 11.6 (선택) Foxglove Studio

Foxglove는 RViz2 대안으로 더 현대적인 ROS2 시각화 도구입니다.

```bash
# Foxglove 설치
sudo snap install foxglove-studio

# 또는 웹 브라우저에서 https://foxglove.dev/ 접속 후 "Open connection"
# → ws://localhost:8765 (ros2 bridge 필요)
```

---

## 11.7 성능 비교: Jetson vs RPi (RViz2)

| 항목 | RPi 4B (Remote PC) | Jetson Orin Nano (네이티브) |
|------|-------------------|--------------------------|
| RViz2 프레임률 | 네트워크 의존 | 60FPS (GPU 가속) |
| LaserScan 표시 | 지연 가능성 | 실시간 |
| 3D 렌더링 | 소프트웨어 렌더링 | GPU 하드웨어 가속 |
| 추가 비용 | Remote PC 필요 | 불필요 |

---

## 📝 연습 문제

> RPi 버전 Day 11과 동일  
> 추가: `jtop`으로 RViz2 실행 중 GPU 사용률을 측정하세요
