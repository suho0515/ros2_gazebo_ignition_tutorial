# ROS2 Humble과 Gazebo Ignition 튜토리얼

이 레포지토리는 ROS2 Humble과 Gazebo Ignition을 함께 사용하는 방법을 설명하는 튜토리얼입니다.

## 목차

1. [환경 설정 및 Gazebo GUI 실행하기](chapter1/chapter1.md)
   - ROS2 Humble 설치
   - Gazebo Ignition 설치
   - SDF 파일 작성
   - Gazebo GUI 실행하기

## 프로젝트 구조

```
.
├── chapter1/           # 1장: 환경 설정
│   ├── chapter1.md     # 튜토리얼 문서
│   └── empty_world.sdf # 예제 SDF 파일
└── README.md           # 프로젝트 설명
```

## 사전 요구사항

- Ubuntu 22.04 LTS
- ROS2 Humble
- Gazebo Ignition

## 시작하기

1. 레포지토리 클론
```bash
git clone https://github.com/username/ros2_gazebo_ignition_tutorial.git
cd ros2_gazebo_ignition_tutorial
```

2. 튜토리얼 문서 따라하기
```bash
# 예: 1장 튜토리얼의 SDF 파일 실행
cd chapter1
ign gazebo -r empty_world.sdf
```