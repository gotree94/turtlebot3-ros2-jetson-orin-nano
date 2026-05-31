# Day 17 — Navigation2 개념

> **RPi 버전 Day 17과 내용이 동일합니다.**
>
> Jetson Orin Nano에서는 Nav2도 **온보드에서 실행**하여
> Remote PC 없이 완전한 자율주행이 가능합니다.

---

## Nav2 이론 요약

| 개념 | 설명 |
|------|------|
| Nav2 | ROS2 Navigation Stack |
| Planner | 전역 경로 계획 (NavFn, Smac) |
| Controller | 지역 경로 추종 (DWB, Regulated Pure Pursuit) |
| AMCL | 적응형 몬테카를로 위치 추정 |
| Behavior Tree | Nav2 동작 트리 (BT) |

## Jetson 적용

- Nav2 실행 위치: **온보드** (RPi는 Remote PC)
- Behavior Tree 파라미터 튜닝: **Jetson에서 직접 수정 후 즉시 재시작**
- AMCL 성능: RPi 대비 **2~3배 빠른 수렴** (더 많은 파티클)

---

> **전체 내용:** `../turtlebot3-ros2-curriculum/week3/day17_navigation2_concepts.md` 참조
>
> 이론 내용: Nav2 아키텍처, Planner/Controller/AMCL/BT 개념 완전 동일
