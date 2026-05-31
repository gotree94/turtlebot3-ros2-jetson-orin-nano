# Week 1 연습 문제 (Jetson Orin Nano)

> **기본 문제:** `../turtlebot3-ros2-curriculum/exercises/week1_exercises.md` 참조
>
> 아래는 Jetson Orin Nano에 특화된 추가 문제입니다.

---

## Jetson 추가 문제

### 문제 1: GPU 정보 수집 노드

Jetson의 GPU 정보를 `/gpu_info` 토픽으로 발행하는 ROS2 노드를 작성하세요.

```python
# 힌트: jetson-stats의 jtop 라이브러리 활용
# pip3 install jetson-stats
from jtop import jtop
```

**요구사항:**
- GPU 사용률 (%) 발행
- GPU 메모리 사용량 발행
- 1초 주기로 업데이트

### 문제 2: CUDA vs CPU 비교

간단한 행렬 연산(1000x1000)을 CPU와 CUDA에서 각각 실행하고
성능을 비교하는 ROS2 서비스를 작성하세요.

```bash
# 서비스 요청
ros2 service call /benchmark std_srvs/srv/Trigger
# 응답: "CPU: 245ms, CUDA: 12ms, Speedup: 20.4x"
```

### 문제 3: 전원 모드에 따른 성능 측정

Jetson의 각 전원 모드(nvpmodel)에서 ROS2 노드 성능을 측정하세요.

| 모드 | 전력 | CPU | GPU | ROS2 성능 |
|------|------|-----|-----|-----------|
| MAXN | 15W | 4 core | 1024 | |
| 7W | 7W | 2 core | 512 | |
| 5W | 5W | 2 core | 300 | |

각 모드에서 turtlesim 노드의 반응 속도를 측정하고 차이를 기록하세요.
