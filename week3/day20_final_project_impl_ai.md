# Day 20 — 최종 프로젝트 구현: AI 객체 탐지 통합

> **목표:** YOLO 객체 탐지를 ROS2에 통합하고, 순찰 시스템과 연결한다.

---

## 20.1 YOLO + ROS2 통합 노드

`~/ros2_ws/src/patrol_bot_ai/patrol_bot_ai/object_detector.py`:

```python
#!/usr/bin/env python3
"""
Jetson Orin Nano AI 객체 탐지 노드
YOLOv8 (nano) + TensorRT 최적화로 실시간 추론
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from std_msgs.msg import String
from visualization_msgs.msg import Marker, MarkerArray
from cv_bridge import CvBridge
import cv2
import numpy as np


class AIObjectDetector(Node):
    """
    GPU 가속 YOLO 객체 탐지 노드
    
    - USB 카메라 /image_raw 구독
    - YOLOv8 Nano 모델로 객체 탐지 (CUDA)
    - 결과를 /detections 토픽으로 발행
    - RViz2 Marker로 시각화
    """

    def __init__(self):
        super().__init__('ai_object_detector')

        # Publisher
        self.detection_pub = self.create_publisher(
            String, '/detections', 10)
        self.marker_pub = self.create_publisher(
            MarkerArray, '/detection_markers', 10)

        # Subscriber (USB 카메라)
        self.sub = self.create_subscription(
            Image, '/image_raw', self.image_callback, 10)
        self.bridge = CvBridge()

        # YOLO 모델 로드 (GPU)
        self.get_logger().info('Loading YOLOv8 model on GPU...')
        try:
            from ultralytics import YOLO
            self.model = YOLO('yolov8n.pt')  # Nano 모델
            self.model.to('cuda')  # GPU 추론
            self.get_logger().info('✅ YOLOv8 loaded on CUDA')
        except Exception as e:
            self.get_logger().error(f'YOLO load failed: {e}')
            self.model = None

        # 탐지 결과 저장용
        self.last_detections = []

        # 추론 성능 측정
        self.inference_times = []
        self.create_timer(30.0, self.report_performance)

        self.get_logger().info('🤖 AI Object Detector started')

    def image_callback(self, msg):
        """카메라 이미지 수신 → YOLO 추론"""
        if self.model is None:
            return

        try:
            # ROS Image → OpenCV
            cv_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
        except Exception as e:
            self.get_logger().error(f'CV bridge error: {e}')
            return

        # GPU 추론
        import time
        t0 = time.time()
        results = self.model(cv_image, device='cuda', verbose=False)
        inference_time = (time.time() - t0) * 1000  # ms
        self.inference_times.append(inference_time)

        # 결과 파싱
        detections = []
        for r in results:
            boxes = r.boxes
            if boxes is not None:
                for box in boxes:
                    x1, y1, x2, y2 = box.xyxy[0].tolist()
                    conf = box.conf[0].item()
                    cls = int(box.cls[0].item())
                    label = self.model.names[cls]

                    if conf > 0.5:  # 신뢰도 임계값
                        detections.append({
                            'label': label,
                            'confidence': conf,
                            'bbox': [x1, y1, x2, y2]
                        })

        self.last_detections = detections

        # 탐지 결과 발행
        if detections:
            msg_out = String()
            msg_out.data = ';'.join([f"{d['label']}:{d['confidence']:.2f}"
                                     for d in detections])
            self.detection_pub.publish(msg_out)

            # Marker 시각화
            self.publish_markers(detections)

    def publish_markers(self, detections):
        """RViz2 MarkerArray로 객체 위치 표시"""
        marker_array = MarkerArray()

        for i, det in enumerate(detections):
            marker = Marker()
            marker.header.frame_id = 'camera_link'
            marker.header.stamp = self.get_clock().now().to_msg()
            marker.ns = 'detections'
            marker.id = i
            marker.type = Marker.TEXT_VIEW_FACING
            marker.action = Marker.ADD

            marker.pose.position.x = 0.5
            marker.pose.position.y = -0.3 + i * 0.3
            marker.pose.position.z = 0.5

            marker.text = f"{det['label']} ({det['confidence']:.2f})"
            marker.scale.z = 0.15

            marker.color.a = 1.0
            marker.color.r = 1.0 if det['confidence'] > 0.7 else 0.0
            marker.color.g = 1.0 if det['confidence'] < 0.7 else 0.0
            marker.color.b = 0.0

            marker_array.markers.append(marker)

        self.marker_pub.publish(marker_array)

    def report_performance(self):
        """30초마다 추론 성능 보고"""
        if self.inference_times:
            avg_time = sum(self.inference_times) / len(self.inference_times)
            fps = 1000.0 / avg_time if avg_time > 0 else 0
            self.get_logger().info(
                f'📊 YOLO Inference: {avg_time:.1f}ms ({fps:.1f} FPS)')
            # 최근 30초치만 유지
            self.inference_times = []

    def get_detections(self):
        """다른 노드에서 탐지 결과 조회용"""
        return self.last_detections

    def destroy_node(self):
        self.report_performance()
        super().destroy_node()


def main(args=None):
    rclpy.init(args=args)
    node = AIObjectDetector()
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

## 20.2 Waypoint + AI 통합 Navigator

`~/ros2_ws/src/patrol_bot_ai/patrol_bot_ai/ai_waypoint_navigator.py`:

```python
#!/usr/bin/env python3
"""
AI 통합 Waypoint Navigator

각 Waypoint 도착 시:
1. Nav2로 목적지 도착
2. 도착 확인 후 AI 객체 탐지 수행
3. 탐지 결과를 로그에 기록
4. 이상 객체 감지 시 경고 발행
"""

import rclpy
from rclpy.node import Node
from rclpy.action import ActionClient
from nav2_msgs.action import NavigateToPose
from std_msgs.msg import String
import yaml
import os
import math


class AIWaypointNavigator(Node):
    """AI 객체 탐지 + Waypoint 순찰"""

    def __init__(self):
        super().__init__('ai_waypoint_navigator')
        self.nav_client = ActionClient(
            self, NavigateToPose, '/navigate_to_pose')

        # Waypoint 로드
        self.waypoints = self.load_waypoints()
        self.current_index = 0

        # AI 탐지 결과 구독
        self.detection_sub = self.create_subscription(
            String, '/detections', self.detection_callback, 10)

        # Waypoint별 탐지 결과 저장
        self.waypoint_log = {}

        self.get_logger().info(
            f'🚀 AI Waypoint Navigator: {len(self.waypoints)} waypoints')

    def load_waypoints(self):
        """YAML waypoint 로드"""
        path = os.path.expanduser('~/patrol_bot_ai/config/waypoints.yaml')
        with open(path, 'r') as f:
            config = yaml.safe_load(f)
        return [{
            'name': w['name'],
            'x': w['position']['x'],
            'y': w['position']['y'],
            'yaw': math.radians(w.get('orientation', {}).get('yaw', 0)),
            'scan_for_objects': w.get('scan_for_objects', True)
        } for w in config['waypoints']]

    def start_patrol(self):
        self.send_next_goal()

    def send_next_goal(self):
        if self.current_index >= len(self.waypoints):
            self.get_logger().info('✅ Patrol complete!')
            self.print_summary()
            return

        wp = self.waypoints[self.current_index]
        self.get_logger().info(f'📍 {wp["name"]}')

        goal = NavigateToPose.Goal()
        goal.pose.header.frame_id = 'map'
        goal.pose.pose.position.x = wp['x']
        goal.pose.pose.position.y = wp['y']

        goal.pose.pose.orientation.z = math.sin(wp['yaw'] / 2)
        goal.pose.pose.orientation.w = math.cos(wp['yaw'] / 2)

        self.nav_client.send_goal_async(
            goal).add_done_callback(self.goal_callback)

    def goal_callback(self, future):
        if future.result().accepted:
            future.result().get_result_async().add_done_callback(
                self.result_callback)

    def result_callback(self, future):
        wp = self.waypoints[self.current_index]

        if wp['scan_for_objects']:
            self.get_logger().info(f'🔍 Scanning for objects at {wp["name"]}...')
            # 3초 대기하며 객체 탐지 결과 수집
            self.create_timer(3.0, lambda: self.finish_waypoint(wp))

        self.current_index += 1
        self.create_timer(1.0, self.send_next_goal)

    def finish_waypoint(self, wp):
        """Waypoint 완료 처리"""
        detections = self.get_detection_summary()
        self.waypoint_log[wp['name']] = detections
        self.get_logger().info(f'📝 {wp["name"]}: {len(detections)} objects')

    def detection_callback(self, msg):
        self.last_detections = msg.data

    def get_detection_summary(self):
        return self.last_detections or 'No detections'

    def print_summary(self):
        for wp_name, detections in self.waypoint_log.items():
            self.get_logger().info(f'{wp_name}: {detections}')


def main(args=None):
    rclpy.init(args=args)
    node = AIWaypointNavigator()
    node.start_patrol()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

---

## 20.3 GPU 추론 성능 목표

| 모델 | GPU 메모리 | 추론 시간 | FPS | Jetson Orin Nano |
|------|-----------|----------|-----|-----------------|
| YOLOv8n | ~1.2GB | 10-15ms | 60-80 | ✅ 실시간 |
| YOLOv8s | ~2.0GB | 20-30ms | 30-50 | ✅ 실시간 |
| YOLOv8m | ~3.5GB | 40-60ms | 15-25 | ✅ 가능 |
| YOLOv8l | ~6.0GB | 70-100ms | 10-15 | ⚠️ 8GB 모델만 |

---

## 20.4 통합 실행

```bash
# 1. 카메라 실행
ros2 run v4l2_camera v4l2_camera_node

# 2. AI 객체 탐지
ros2 run patrol_bot_ai object_detector

# 3. AI Waypoint Navigator
ros2 run patrol_bot_ai ai_waypoint_navigator

# 4. RViz2
rviz2
```

---

## 📝 연습 문제

1. YOLOv8n 모델을 TensorRT로 최적화하여 추론 속도를 측정하세요
2. 각 Waypoint에서 탐지된 객체를 CSV 파일로 저장하는 기능을 추가하세요
3. 특정 객체(사람, 가방) 탐지 시 순찰을 일시 중지하고 집중 스캔하는 로직을 구현하세요
