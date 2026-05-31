# Week 2 연습 문제 (Jetson Orin Nano)

> **기본 문제:** `../turtlebot3-ros2-curriculum/exercises/week2_exercises.md` 참조
>
> 아래는 Jetson Orin Nano에 특화된 추가 문제입니다.

---

## Jetson 추가 문제

### 문제 1: GPU 가속 LIDAR 전처리

LIDAR `/scan` 데이터를 GPU에서 전처리하는 노드를 설계하세요.

**옵션:**
- CuPy를 사용한 LIDAR 데이터 GPU 필터링
- 이상치 제거를 GPU 벡터 연산으로 처리
- 결과를 `/filtered_scan`으로 발행

```python
# 힌트:
import cupy as cp
import numpy as np

# LIDAR 데이터를 GPU로 전송
ranges_gpu = cp.asarray(ranges_np)
# GPU에서 필터링
filtered = cp.where(ranges_gpu > 0.1, ranges_gpu, 0)
```

### 문제 2: 온보드 성능 로깅

TurtleBot3 Bringup + RViz2 동시 실행 시 시스템 리소스를 기록하는
대시보드 노드를 작성하세요.

```bash
# 출력 예시 (1초마다)
# [PERF] CPU: 45% | GPU: 23% | RAM: 3.2G | Temp: 52°C
```

**팁:** `tegrastats`, `jtop`, `psutil` 라이브러리 활용

### 문제 3: 센서 데이터 병렬 처리

LIDAR, IMU, Wheel Odometry 데이터를 GPU에서 병렬 처리하는
파이프라인을 설계하세요. (개념 설계 + 의사 코드)
