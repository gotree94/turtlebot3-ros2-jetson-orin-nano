# Day 14 — 미니 프로젝트: LIDAR 장애물 회피

> **RPi 버전 Day 14와 내용이 동일합니다.**
>
> Jetson Orin Nano에서는 모든 프로세스를 **온보드에서 단독 실행**하며,
> GPU 가속으로 더 복잡한 전처리도 가능합니다.

---

## Jetson 온보드 실행

```bash
# 터미널 1 (tmux 창 0): Bringup
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_bringup robot.launch.py

# 터미널 2 (tmux 창 1): 장애물 회피 노드
ros2 run my_obstacle_avoid obstacle_avoidance

# 터미널 3 (tmux 창 2): 모니터링
ros2 topic echo /avoid_status
```
---

> **전체 내용:** `../turtlebot3-ros2-curriculum/week2/day14_mini_project_obstacle.md` 참조
>
> 실습: LIDAR 기반 장애물 감지, 회피 알고리즘, 체크리스트 완전 동일
>
> **확장 과제 (Jetson):** LIDAR 데이터 + GPU 가속 장애물 분류로 회피 성능 향상
