# Day 13 — TurtleBot3 URDF & RViz2 시각화

> **RPi 버전 Day 13과 내용이 동일합니다.**
>
> **핵심 차별점:** Jetson Orin Nano는 RViz2를 **온보드에서 GPU 가속으로 실행**합니다.
> Remote PC가 필요 없으며, 부드러운 3D 렌더링이 가능합니다.

---

## Jetson 온보드 RViz2 실행

```bash
# TurtleBot3 모델 확인
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_bringup robot.launch.py

# 새 터미널에서 RViz2 실행 (Jetson 모니터에서!)
rviz2

# RViz2 설정:
# 1. Fixed Frame → base_footprint 또는 odom
# 2. Add → RobotModel → robot_description
# 3. Add → TF
# 4. 저장: File → Save Config → ~/tb3.rviz
```

## RPi 대비 장점

| 항목 | RPi 4B | Jetson Orin Nano |
|------|--------|-------------------|
| RViz2 실행 | Remote PC | **온보드 GPU 가속** |
| 렌더링 품질 | Remote PC 의존 | **네이티브 3D 가속** |
| 새로고침 | WiFi 지연 | **즉시** |

---

> **전체 내용:** `../turtlebot3-ros2-curriculum/week2/day13_rviz2_urdf.md` 참조
>
> 실습: URDF 로드, TF 트리 확인, Link/Joint 시각화 완전 동일
