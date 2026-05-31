# Day 7 — 미니 프로젝트: GPU/CPU 온도 모니터링 (Jetson Edition)

> **목표:** Jetson Orin Nano의 CPU + GPU 온도를 동시 모니터링하는 ROS2 노드를 만든다.

---

## 7.1 프로젝트 개요

Jetson Orin Nano는 CPU와 GPU의 온도를 각각 읽을 수 있습니다.
이 데이터를 ROS2 토픽으로 발행하고, 실시간으로 모니터링하며 과열을 경고하는 시스템을 구축합니다.

**RPi 버전과의 차이점:**
- ✅ GPU 온도 추가 모니터링 (RPi는 GPU 없음)
- ✅ GPU 클럭 정보 포함
- ✅ 전력 소비 정보 포함 (선택)

---

## 7.2 Jetson 온도 센서 구조

```bash
# 모든 온도 존 확인
cat /sys/devices/virtual/thermal/thermal_zone*/type
cat /sys/devices/virtual/thermal/thermal_zone*/temp

# 일반적인 맵핑:
# thermal_zone0: CPU (AO-therm)
# thermal_zone1: GPU (GPU-therm)
# thermal_zone2: 보드 (aux-therm)
# thermal_zone3: 전원 모듈 (POM-therm)

# GPU 클럭 확인
cat /sys/kernel/debug/clk/gpu/enable
cat /sys/kernel/debug/clk/gpu/rate

# 전력 소비 (선택)
cat /sys/bus/i2c/drivers/ina3221/1-0040/hwmon/hwmon*/power1_input
```

---

## 7.3 Jetson 온도 Publisher 노드

`~/ros2_ws/src/temp_monitor_jetson/temp_monitor_jetson/jetson_temp_publisher.py`:

```python
#!/usr/bin/env python3
"""
Jetson Orin Nano CPU/GPU 온도 모니터링 ROS2 Publisher
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64, String
import subprocess
import re
import os


class JetsonTempPublisher(Node):
    """Jetson CPU + GPU 온도 발행 노드"""

    # Jetson Orin Nano 온도 존 맵핑
    THERMAL_ZONES = {
        'cpu': {'index': 0, 'name': 'CPU'},
        'gpu': {'index': 1, 'name': 'GPU'},
        'board': {'index': 2, 'name': 'Board'},
    }

    def __init__(self):
        super().__init__('jetson_temp_publisher')

        # Publishers
        self.pub_cpu = self.create_publisher(Float64, '/temp/cpu', 10)
        self.pub_gpu = self.create_publisher(Float64, '/temp/gpu', 10)
        self.pub_board = self.create_publisher(Float64, '/temp/board', 10)
        self.pub_status = self.create_publisher(String, '/temp/status', 10)

        # GPIO 팬 제어 (선택)
        self.fan_pwm_path = '/sys/devices/platform/pwm-fan/hwmon/hwmon*/pwm1'

        # 1초 타이머
        self.create_timer(1.0, self.publish_temps)

        # 임계값
        self.cpu_warn = 75.0
        self.gpu_warn = 70.0
        self.cpu_crit = 85.0
        self.gpu_crit = 80.0

        self.get_logger().info('🌡️  Jetson Temperature Publisher Started')

    def read_thermal_zone(self, zone_index):
        """온도 존에서 온도 읽기 (millidegree → Celsius)"""
        try:
            path = f'/sys/devices/virtual/thermal/thermal_zone{zone_index}/temp'
            with open(path, 'r') as f:
                return float(f.read().strip()) / 1000.0
        except Exception as e:
            self.get_logger().debug(f'Failed to read zone {zone_index}: {e}')
            return None

    def read_gpu_clock(self):
        """GPU 클럭 속도 읽기 (MHz)"""
        try:
            result = subprocess.run(
                ['cat', '/sys/kernel/debug/clk/gpu/rate'],
                capture_output=True, text=True, timeout=1
            )
            if result.returncode == 0:
                return int(result.stdout.strip()) // 1000000  # Hz → MHz
        except:
            pass
        return None

    def control_fan(self, cpu_temp, gpu_temp):
        """온도 기반 PWM 팬 속도 제어 (0~255)"""
        max_temp = max(cpu_temp or 0, gpu_temp or 0)
        try:
            # glob으로 pwm1 경로 찾기
            import glob
            pwm_files = glob.glob('/sys/devices/platform/pwm-fan/hwmon/hwmon*/pwm1')
            for pwm_file in pwm_files:
                if max_temp > 70:
                    duty = 255  # 최대 속도
                elif max_temp > 60:
                    duty = 180  # 중간 속도
                elif max_temp > 50:
                    duty = 100  # 저속
                else:
                    duty = 60   # 최소
                # 10초 주기로만 업데이트 (과도한 쓰기 방지)
                import random
                if random.random() < 0.1:  # 10% 확률로 업데이트
                    with open(pwm_file, 'w') as f:
                        f.write(str(duty))
        except:
            pass

    def publish_temps(self):
        """주기적 온도 발행"""
        cpu_temp = self.read_thermal_zone(0)
        gpu_temp = self.read_thermal_zone(1)
        board_temp = self.read_thermal_zone(2)
        gpu_clock = self.read_gpu_clock()

        # 발행
        now = self.get_clock().now()

        if cpu_temp is not None:
            msg = Float64()
            msg.data = cpu_temp
            self.pub_cpu.publish(msg)

        if gpu_temp is not None:
            msg = Float64()
            msg.data = gpu_temp
            self.pub_gpu.publish(msg)

        if board_temp is not None:
            msg = Float64()
            msg.data = board_temp
            self.pub_board.publish(msg)

        # 상태 메시지
        status = f'CPU:{cpu_temp:.1f}°C | GPU:{gpu_temp:.1f}°C | '
        status += f'Board:{board_temp:.1f}°C'
        if gpu_clock:
            status += f' | GPU Clock:{gpu_clock}MHz'

        status_msg = String()
        status_msg.data = status
        self.pub_status.publish(status_msg)

        # 경고 체크
        warnings = []
        if cpu_temp and cpu_temp > self.cpu_crit:
            warnings.append(f'🔥 CPU CRITICAL: {cpu_temp:.1f}°C')
        elif cpu_temp and cpu_temp > self.cpu_warn:
            warnings.append(f'⚠️  CPU Warning: {cpu_temp:.1f}°C')

        if gpu_temp and gpu_temp > self.gpu_crit:
            warnings.append(f'🔥 GPU CRITICAL: {gpu_temp:.1f}°C')
        elif gpu_temp and gpu_temp > self.gpu_warn:
            warnings.append(f'⚠️  GPU Warning: {gpu_temp:.1f}°C')

        for w in warnings:
            self.get_logger().warn(w)

        # 팬 제어
        self.control_fan(cpu_temp, gpu_temp)


def main(args=None):
    rclpy.init(args=args)
    node = JetsonTempPublisher()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

---

## 7.4 실행 & 확인

```bash
# 패키지 빌드
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash

# 노드 실행
ros2 run temp_monitor_jetson jetson_temp_publisher

# 토픽 확인
ros2 topic list | grep /temp
ros2 topic echo /temp/status

# 그래프
rqt_plot /temp/cpu /temp/gpu
```

---

## 7.5 부하 테스트

```bash
# GPU 부하
sudo apt install -y cuda-samples
cd /usr/local/cuda/samples/0_Simple/vectorAdd
make
./vectorAdd &

# CPU 부하
sudo apt install -y stress
stress --cpu 6 --timeout 30 &

# 온도 변화 관찰
rqt_plot /temp/cpu /temp/gpu
```

---

## 📝 연습 문제

> RPi 버전 Day 7의 문제 + 추가:
1. GPU 온도와 GPU 클럭 속도의 상관관계를 측정하세요
2. MAXN 전원 모드와 15W 모드에서 각각 부하 테스트를 수행하고 온도 차이를 기록하세요
3. 온도 데이터를 주제로 CPU vs GPU 중 어느 쪽이 먼저 과열되는지 분석하세요
