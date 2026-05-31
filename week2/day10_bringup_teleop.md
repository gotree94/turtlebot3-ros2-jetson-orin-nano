# Day 10 — TurtleBot3 Bringup & Teleoperation

> **RPi 버전 Day 10과 완전히 동일합니다.**

Bringup 및 Teleoperation 과정은 동일합니다.  
Jetson Orin Nano의 장점은 **모든 터미널을 한 시스템에서 tmux로 관리**할 수 있다는 점입니다.

```bash
# tmux 세션 시작
tmux new -s tb3

# 창 분할: Bringup + Teleop
Ctrl+B %  # 세로 분할
Ctrl+B "  # 가로 분할

# 각 창에서:
# 좌측: ros2 launch turtlebot3_bringup robot.launch.py
# 우측 상단: ros2 run turtlebot3_teleop teleop_keyboard
# 우측 하단: rviz2 또는 ros2 topic echo /scan
```

> **상세 내용:** `../turtlebot3-ros2-curriculum/week2/day10_bringup_teleop.md` 참조
