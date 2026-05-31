# Week 3 — SLAM, Navigation & 최종 프로젝트 (Jetson Orin Nano)

> **목표:** SLAM으로 환경 지도를 생성하고, Navigation2로 자율주행하며, AI 기반 최종 프로젝트를 완성한다.

---

## 📅 주간 일정

| 일차 | 주제 | RPi 버전 대비 |
|------|------|-------------|
| Day 15 | SLAM 이론 | ✅ 동일 |
| Day 16 | SLAM 매핑 — 온보드 실행 | 🔶 Jetson 온보드 SLAM (Remote PC 불필요) |
| Day 17 | Navigation2 개념 | ✅ 동일 |
| Day 18 | Nav2 자율주행 — 온보드 실행 | 🔶 Jetson 온보드 Nav2 |
| Day 19 | 최종 프로젝트 기획 | 🔶 AI 기반 순찰 로봇 설계 |
| Day 20 | 최종 프로젝트 구현 — AI 통합 | 🔶 YOLO 객체 탐지 + 순찰 시스템 |
| Day 21 | 최종 프로젝트 완성 | 🔶 GPU 가속 최종 데모 |

---

## 🔑 Week 3의 핵심 차별점

| 항목 | RPi 4B | Jetson Orin Nano |
|------|--------|-------------------|
| SLAM 실행 위치 | Remote PC | **Jetson 온보드** |
| Nav2 실행 위치 | Remote PC | **Jetson 온보드** |
| 지도 저장 위치 | Remote PC | **Jetson 로컬** |
| 객체 탐지 | 불가능 | **YOLO 실시간 (GPU 가속)** |
| 최종 프로젝트 | 단순 순찰 | **AI 순찰 (객체 인식 + 자율주행)** |

> 💡 **모든 Nav2/SLAM 작업을 Jetson 단독으로 처리할 수 있습니다.**  
> Remote PC는 모니터링 용도로만 선택적으로 사용하세요.
