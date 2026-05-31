# Day 15 — SLAM 이론

> **RPi 버전 Day 15와 내용이 동일합니다.**

---

## SLAM 이론 요약

| 개념 | 설명 |
|------|------|
| SLAM | Simultaneous Localization and Mapping |
| EKF SLAM | 확장 칼만 필터 기반 |
| GraphSLAM | 그래프 최적화 기반 (slam_toolbox 사용) |
| Landmark | 랜드마크 기반 vs LIDAR 기반 |

## Jetson 차별점

- SLAM 실행 위치: **온보드 Jetson** (RPi는 Remote PC 필요)
- 지도 저장: **Jetson 로컬 파일시스템**
- GPU 활용: SLAM 자체는 CPU 위주지만, 추후 **Isaac ROS GPU SLAM**으로 확장 가능

---

> **전체 내용:** `../turtlebot3-ros2-curriculum/week3/day15_slam_theory.md` 참조
>
> 이론 내용: SLAM 정의, EKF vs GraphSLAM, slam_toolbox 아키텍처, 매핑 과정 완전 동일
