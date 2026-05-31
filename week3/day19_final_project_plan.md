# Day 19 — 최종 프로젝트 기획: AI 자율 순찰 로봇

> **목표:** Jetson Orin Nano의 GPU를 활용한 AI 기반 자율 순찰 로봇을 설계한다.

---

## 19.1 프로젝트 개요

**프로젝트명:** TurtleBot3 AI 자율 순찰 로봇 (AI Patrol Bot)

**Jetson만의 차별점:**
- ✅ **YOLO 실시간 객체 탐지** (GPU 가속)
- ✅ **온보드 SLAM + Nav2** (Remote PC 불필요)
- ✅ **GPU 가속 센서 전처리** (선택)
- ✅ **완전 자율주행 + AI 인식**

---

## 19.2 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│                 Jetson Orin Nano (모든 것 온보드)         │
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ YOLO        │  │ Waypoint    │  │ Nav2 Stack      │  │
│  │ Object      │  │ Navigator   │  │ (Planner+AMCL+  │  │
│  │ Detection   │  │             │  │  Controller)    │  │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘  │
│         │                │                  │           │
│         └────────────────┼──────────────────┘           │
│                          │                              │
│  ┌───────────────────────▼──────────────────────────┐   │
│  │              Patrol Monitor                      │   │
│  │  (로그 + 통계 + 알림 + 객체 탐지 결과 기록)       │   │
│  └───────────────────────┬──────────────────────────┘   │
│                          │                              │
│  ┌───────────────────────▼──────────────────────────┐   │
│  │              RViz2 (GPU 가속 시각화)              │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                              │
└──────────────────────────┼──────────────────────────────┘
                           │ ROS_DOMAIN_ID=30
                           │ (WiFi)
┌──────────────────────────▼──────────────────────────────┐
│                   TurtleBot3 Burger                     │
│  (OpenCR + LDS + DYNAMIXEL XL430-W250 x2)              │
└─────────────────────────────────────────────────────────┘
```

---

## 19.3 추가 기능: 객체 탐지

Jetson Orin Nano의 GPU로 실시간 객체 탐지를 순찰 시스템에 통합:

```python
# 의사 코드: 객체 탐지 노드
class ObjectDetectionNode(Node):
    def __init__(self):
        # YOLOv5/v8 모델 로드 (TensorRT 최적화)
        self.model = torch.hub.load('ultralytics/yolov8', 'nano')
        self.model.to('cuda')  # GPU 추론
        self.camera_sub = create_subscription(Image, '/image_raw', ...)
        self.marker_pub = create_publisher(Marker, '/detected_objects', ...)

    def detect_objects(self, image):
        results = self.model(image)
        # Waypoint 도착 시: "사람 감지됨", "물체 감지됨" 로그
        # 각 Waypoint에서 정지 이미지 캡처
        # 이상 상황(사람 없음, 침입자) 알림
```

---

## 19.4 개발 일정

### Day 19: 기획 (오늘)
- [ ] 시스템 아키텍처 설계
- [ ] YOLO 설치 및 GPU 추론 테스트
- [ ] Waypoint 파일 작성
- [ ] 객체 탐지 + 순찰 통합 설계

### Day 20: 구현
- [ ] YOLO + ROS2 통합 노드 구현
- [ ] Waypoint Navigator + AI 통합
- [ ] GPU 가속 객체 탐지 테스트
- [ ] RViz2 마커로 객체 위치 표시

### Day 21: 완성
- [ ] 전체 통합 및 데모
- [ ] 순찰 → 객체 탐지 → 로깅 자동화
- [ ] 최종 시연

---

## 19.5 준비

```bash
# YOLO 의존성
sudo apt install -y python3-pip
pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu121
pip3 install ultralytics

# ROS2 카메라 패키지
sudo apt install -y ros-humble-v4l2-camera ros-humble-cv-bridge
```

---

## 📝 기획 문제

> RPi 버전 Day 19의 기획 문제 + 추가:
1. YOLO로 감지할 객체 5가지를 선정하고 각각에 대한 알림 수준을 정의하세요
2. GPU 추론 시간을 측정하고 ROS2 주기(10Hz)에 적합한지 확인하세요
3. 객체 탐지 결과를 RViz2 Marker로 표시하는 방법을 설계하세요
