# Day 9 — Remote PC TurtleBot3 패키지 (선택)

> **RPi 버전 Day 9와 완전히 동일합니다.**

Jetson Orin Nano는 자체 성능이 충분하므로 Remote PC가 필요하지 않습니다.
필요 시 아래 명령어로 PC에 설치하세요:

```bash
sudo apt install -y ros-humble-turtlebot3*
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30
echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
```

> **상세 내용:** `../turtlebot3-ros2-curriculum/week2/day09_remote_pc_tb3_pkg.md` 참조
