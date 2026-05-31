# Week 3 연습 문제 (Jetson Orin Nano)

> **기본 문제:** `../turtlebot3-ros2-curriculum/exercises/week3_exercises.md` 참조
>
> 아래는 Jetson Orin Nano에 특화된 추가 문제입니다.

---

## Jetson 추가 문제

### 문제 1: TensorRT 최적화된 SLAM

slam_toolbox의 파라미터 중 Jetson에서 더 좋은 성능을 내기 위해
조정할 수 있는 설정을 찾고 테스트하세요.

```yaml
# 테스트할 파라미터 예시
slam_params:
  minimum_travel_distance: 0.1  # 기본 0.5 → 더 정밀한 매핑
  minimum_travel_heading: 0.1   # 기본 0.5
  resolution: 0.05              # 지도 해상도 (기본 0.05)
```

**과제:** RPi 기준 파라미터와 비교하여 Jetson에서 개선된 점을 측정하세요.

### 문제 2: YOLO + Nav2 통합

객체 탐지 결과를 Nav2 경로 계획에 반영하는 시스템을 설계하세요.

```python
# 힌트: 특정 객체 감지 시 해당 영역을 costmap에 반영
class AIEnhancedPlanner:
    def __init__(self):
        # Nav2 costmap 업데이트
        self.costmap_pub = self.create_publisher(
            OccupancyGrid, '/ai_costmap_update', 10)

    def object_detected(self, x, y, label):
        if label in ['person', 'dog']:  # 회피할 객체
            self.add_costmap_obstacle(x, y, radius=1.0)
```

### 문제 3: GPU 가속 시각화

OpenCV + CUDA를 사용하여 실시간 카메라 영상에 객체 탐지 결과를
오버레이하는 GPU 가속 영상 처리 노드를 구현하세요.

```python
# 힌트: cv2.cuda_GpuMat 활용
gpu_frame = cv2.cuda_GpuMat()
gpu_frame.upload(cv_image)
# GPU에서 바로 처리
gpu_gray = cv2.cuda.cvtColor(gpu_frame, cv2.COLOR_BGR2GRAY)
```

### 문제 4: 멀티 모델 추론

YOLOv8n(빠름)과 YOLOv8m(정확함)을 상황에 따라 전환하는
적응형 객체 탐지 시스템을 설계하세요.

| 상황 | 모델 | 목표 |
|------|------|------|
| 이동 중 | YOLOv8n | 빠른 탐지 (60FPS) |
| 정지 후 | YOLOv8m | 정밀 탐지 (20FPS) |
