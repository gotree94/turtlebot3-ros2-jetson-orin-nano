# Day 5 — 첫 번째 ROS2 패키지

> **RPi 버전 Day 5와 내용이 동일합니다.**
>
> Jetson Orin Nano의 **NVMe SSD + 강력한 CPU** 덕분에
> `colcon build` 시간이 **5~10초**로 RPi(1~2분) 대비 획기적으로 빠릅니다.

---

## 5.1 Jetson 빌드 성능

```bash
# RPi 4B: ~60초
# Jetson Orin Nano: ~5초
time colcon build --symlink-install

# real 0m4.823s  ← Jetson 기준
```

## 5.2 실습 참고

- 패키지 생성, pub/sub 노드 작성, 빌드, 실행 과정은 **RPi 버전과 완전 동일**
- Jetson에서는 `ros2 run`을 SSH 없이 **로컬 터미널**에서 직접 실행
- `rclcpp` 예제도 동일하게 빌드/실행 가능

---

> **전체 내용:** `../turtlebot3-ros2-curriculum/week1/day05_first_package.md` 참조
>
> 실습: Python publisher/subscriber 노드 작성, setup.py/py(~).cfg 설정, colcon build, ros2 run
