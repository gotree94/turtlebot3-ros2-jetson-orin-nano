# Day 12 — PID 속도 제어 기초

> **RPi 버전 Day 12와 내용이 동일합니다.**
>
> Jetson Orin Nano에서는 더 빠른 디버깅/재빌드가 가능하여
> PID 게인 튜닝 과정이 더 효율적입니다.

---

## Jetson 팁

```bash
# PID 게인 수정 후 재빌드 (5초)
colcon build --packages-select my_pid_controller
ros2 run my_pid_controller velocity_controller

# 실시간 데이터 수집 (동일)
ros2 topic echo /pid_error
ros2 topic echo /cmd_vel
```

> **전체 내용:** `../turtlebot3-ros2-curriculum/week2/day12_pid_control_basics.md` 참조
>
> 실습: PID 이론, Go-to-Goal 노드 작성, 게인 튜닝, 에러 시각화 완전 동일
