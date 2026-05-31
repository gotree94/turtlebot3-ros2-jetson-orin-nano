# Day 4 — ROS2 핵심 개념

> **RPi 버전 Day 4와 내용이 동일합니다.**
>
> Jetson Orin Nano의 경우 모든 예제를 **온보드에서 직접 실행**할 수 있습니다.
> 멀티 터미널이 필요한 실습은 **tmux**를 활용하세요.

---

## 4.1 실습 명령어 정리

```bash
# 터미널 분할 (tmux)
tmux new -s ros2_practice
Ctrl+B %        # 세로 분할
Ctrl+B "        # 가로 분할

# 각 창에서:
# 창 1: turtlesim, rqt_graph, echo 등
# 창 2: teleop 또는 topic pub
```

## 4.2 Jetson 참고 사항

| 항목 | RPi 4B | Jetson Orin Nano |
|------|--------|-------------------|
| turtlesim 실행 | Remote PC | **온보드** |
| rviz2 실행 | Remote PC | **온보드 GPU 가속** |
| rqt_graph 실행 | Remote PC | **온보드** |
| 멀티 터미널 | SSH 다중 | **tmux 단일 시스템** |

---

> **전체 내용:** `../turtlebot3-ros2-curriculum/week1/day04_ros2_concepts.md` 참조
>
> 실습 내용: Topic, Service, Action 통신, pub/sub 패키지 실행, rqt_graph 시각화 완전 동일
