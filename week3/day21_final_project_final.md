# Day 21 — 최종 프로젝트 완성 & 데모

> **목표:** AI 기반 자율 순찰 시스템을 완성하고, Jetson Orin Nano에서 전체 데모를 진행한다.

---

## 21.1 최종 체크리스트

### 하드웨어

```
□ Jetson Orin Nano + NVMe SSD 장착 확인
□ TurtleBot3 Burger 조립 + 배터리 충전
□ OpenCR, LDS USB 연결
□ USB 카메라 장착 (TurtleBot3 Burger 카메라 마운트)
□ 방열판 + 쿨링팬 작동 확인
□ Jetson 전원 모드: MAXN (최대 성능)
```

### 소프트웨어

```
□ JetPack 6.x (Ubuntu 22.04 aarch64)
□ ROS2 Humble desktop + TurtleBot3 패키지
□ CUDA 12.x, cuDNN, TensorRT
□ YOLOv8 (ultralytics) GPU 동작 확인
□ SLAM 지도 생성 완료 (~/jetson_map.yaml)
□ Waypoint YAML 파일 최종 확정
□ patrol_bot_ai 패키지 빌드 완료
```

### 데모 준비

```
□ 데모 공간 장애물 배치
□ 조명 조건 확인 (카메라 인식)
□ WiFi 신호 세기 확인
□ 배터리 완충 (최소 30분 데모 가능)
□ 데모 대본 작성
```

---

## 21.2 최종 데모 시나리오

### 데모 1: 실시간 객체 탐지 (GPU 가속)

```
목표: Jetson GPU로 YOLO 객체 탐지 성능 시연

절차:
  1. USB 카메라 실행
     ros2 run v4l2_camera v4l2_camera_node
  
  2. AI 객체 탐지 시작
     ros2 run patrol_bot_ai object_detector
  
  3. RViz2에서 MarkerArray 표시
     - Add → MarkerArray → Topic: /detection_markers
     - Add → Image → Topic: /image_raw
  
  4. 카메라 앞에 다양한 물체 배치 (병, 책, 컵 등)
  5. GPU 추론 FPS를 로그로 확인
     
예상 결과:
  - YOLOv8n: ~60 FPS
  - 탐지된 객체가 RViz2에 텍스트 마커로 표시
```

### 데모 2: AI 자율 순찰

```
목표: Waypoint 순찰 + 객체 탐지 통합 시스템

절차:
  1. TurtleBot3 Bringup
  2. SLAM 지도 로드 + Nav2
  3. AI 객체 탐지 노드 실행
  4. AI Waypoint Navigator 실행
  5. 순찰 시작 (3개 이상 Waypoint)
  6. 각 Waypoint에서 자동 객체 탐지 및 로깅
  7. 순찰 완료 후 요약 보고서 출력

예상 결과:
  - 순찰 중 각 Waypoint 도착 시 "🔍 Scanning..."
  - 탐지된 객체가 로그에 기록
  - 순찰 완료 후 전체 요약 출력
```

### 데모 3: GPU 성능 (선택)

```
목표: Jetson GPU 활용도 시각화

절차:
  1. jtop 실행 (GPU 사용률 실시간 표시)
  2. 모든 ROS2 노드 실행 상태에서 GPU 사용률 측정
     - RViz2: GPU 10~20%
     - YOLO 추론: GPU 30~50%
     - Nav2: CPU 위주
     - 총 GPU 사용률: 40~70%
```

---

## 21.3 최종 실행 스크립트

`~/patrol_bot_ai/start_ai_patrol.sh`:

```bash
#!/bin/bash
# Jetson Orin Nano AI 순찰 시작 스크립트

set -e

echo "╔══════════════════════════════════════╗"
echo "║  TurtleBot3 AI Patrol - Jetson      ║"
echo "╚══════════════════════════════════════╝"

export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30

# tmux 세션 시작
tmux new-session -d -s ai_patrol -n bringup

# 창 0: Bringup
tmux send-keys -t ai_patrol:bringup \
  "ros2 launch turtlebot3_bringup robot.launch.py" Enter

sleep 3

# 창 1: Nav2
tmux new-window -t ai_patrol -n nav2
tmux send-keys -t ai_patrol:nav2 \
  "ros2 launch turtlebot3_navigation2 navigation2.launch.py \
  map:=$HOME/jetson_map.yaml use_sim_time:=false" Enter

sleep 5

# 창 2: Camera + AI
tmux new-window -t ai_patrol -n ai
tmux send-keys -t ai_patrol:ai \
  "ros2 run v4l2_camera v4l2_camera_node &" Enter
sleep 1
tmux send-keys -t ai_patrol:ai \
  "ros2 run patrol_bot_ai object_detector" Enter

# 창 3: Waypoint Navigator
tmux new-window -t ai_patrol -n patrol
tmux send-keys -t ai_patrol:patrol \
  "ros2 run patrol_bot_ai ai_waypoint_navigator" Enter

# 창 4: RViz2 (GUI)
# 별도로 수동 실행 권장: rviz2

echo ""
echo "✅ All nodes started!"
echo ""
echo "tmux 세션 관리:"
echo "  tmux attach -t ai_patrol    # 세션 접속"
echo "  Ctrl+B 방향키              # 창 이동"
echo "  Ctrl+B 숫자(0-4)           # 특정 창 이동"
echo ""
echo "RViz2는 별도 터미널에서 실행: rviz2"
```

---

## 21.4 예상 GPU 성능

```
Jetson Orin Nano (15W MAXN 모드) - 전체 시스템 가동 시:

프로세스              CPU    GPU    메모리
─────────────────────────────────────────
Bringup (LDS+OpenCR)   8%     -      200MB
Nav2 Stack            25%    10%    800MB
YOLOv8n 추론          10%    45%   1500MB
RViz2                  5%    15%    500MB
Waypoint Navigator     3%     -     100MB
OS + 기타             10%     5%    2000MB
─────────────────────────────────────────
합계                  61%    75%    ~5.1GB

여유: CPU 39%, GPU 25%, RAM ~3GB (8GB 모델 기준)
→ 최종 프로젝트에 여유 리소스 충분!
```

---

## 21.5 문제 해결

| 문제 | 해결 |
|------|------|
| YOLO가 GPU를 사용 안 함 | `model.to('cuda')` 확인, `torch.cuda.is_available()` |
| 카메라 노드 없음 | `ls /dev/video*` 확인, `v4l2_camera` 패키지 설치 확인 |
| GPU 메모리 부족 | YOLO 모델을 nano(s)로 변경, 배치 사이즈 1 확인 |
| RViz2 느림 | Resolution 낮추기, PointCloud2 display 제거 |
| tmux 세션 정리 | `tmux kill-session -t ai_patrol` |

---

## 21.6 최종 회고

### 이 과정에서 배운 것 (Jetson 특화)

```
✅ NVIDIA JetPack 6 + Ubuntu 22.04 aarch64 설정
✅ CUDA, cuDNN, TensorRT 개발 환경 구축
✅ Jetson 전원 모드 및 성능 최적화
✅ GPU 가속 RViz2 네이티브 (Remote PC 불필요)
✅ YOLOv8 GPU 실시간 객체 탐지 + ROS2 통합
✅ 온보드 SLAM + Nav2 자율주행
✅ AI + 자율 순찰 통합 시스템
```

### RPi 버전과의 차이점 요약

| 항목 | RPi 4B | Jetson Orin Nano |
|------|--------|-------------------|
| OS 설치 | Raspberry Pi Imager | SDK Manager / 직접 플래싱 |
| 빌드 시간 | 40-60분 | 3-5분 |
| SLAM/Nav2 | Remote PC 필요 | 온보드 실행 |
| GPU 가속 | 없음 | CUDA 1024 cores |
| 객체 탐지 | 불가능 | YOLO 실시간 (60FPS) |
| RViz2 | Remote PC | 네이티브 GPU 가속 |
| 최종 프로젝트 | 단순 순찰 | AI 기반 순찰 |

---

## 21.7 다음 단계

1. **Isaac ROS** 도입: GPU 가속 Visual SLAM, DNN 추론 파이프라인
2. **멀티 로봇**: 여러 Jetson TurtleBot3 협업 시스템
3. **클라우드 연동**: AWS IoT Greengrass + ROS2
4. **강화 학습**: NVIDIA Isaac Gym + ROS2 연동
5. **엣지 AI 최적화**: TensorRT 양자화(INT8)로 더 빠른 추론

---

> **🎉 Congratulations! You've built an AI-powered autonomous patrol robot with Jetson Orin Nano!**
> 
> 이제 당신은 엣지 AI 로봇 공학의 첫걸음을 떼었습니다.
> Jetson의 GPU 성능을 활용하여 더 복잡하고 지능적인 로봇 시스템을 만들어보세요!
