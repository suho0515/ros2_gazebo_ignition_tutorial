# ROS2 Humble과 Gazebo Ignition 튜토리얼

이 레포지토리는 ROS2 Humble과 Gazebo Ignition을 함께 사용하는 방법을 설명하는 튜토리얼입니다.

## 목차

1. [환경 설정 및 Gazebo GUI 실행하기](chapter1/chapter1.md)
   - ROS2 Humble 설치
   - Gazebo Ignition 설치
   - SDF 파일 작성
   - Gazebo GUI 실행하기

2. [객체 모델링 및 월드에 추가하기](chapter2/chapter2.md)
   - SDF로 3D 객체 모델링
   - 객체를 월드에 포함하기
   - 시뮬레이션 실행하기

3. [모듈식 SDF 파일 구조화](chapter3/chapter3.md)
   - 효율적인 SDF 구성 방법

## 프로젝트 구조

```
.
├── README.md                      # 프로젝트 설명
├── chapter1.md                    # 1장 튜토리얼 문서
├── empty_world.sdf                # 1장에서 사용하는 빈 월드 파일
├── chapter2.md                    # 2장 튜토리얼 문서
├── box.sdf                        # 2장에서 사용하는 기본 객체 모델
├── sphere.sdf                     # 기본 객체 모델
├── world_with_objects.sdf         # 2장 월드 파일
└── chapter3/                      # 3장 관련 파일
    ├── chapter3.md                # 3장 튜토리얼 문서
    ├── physics/                   # 물리 시뮬레이션 설정
    │   └── physics.sdf            # 물리 설정 파일
    ├── plugin/                    # 플러그인 설정
    │   └── plugins.sdf            # 플러그인 설정 파일
    ├── light/                     # 조명 설정
    │   └── lights.sdf             # 조명 설정 파일
    ├── model/                     # 모델 파일
    │   ├── ground_plane.sdf       # 지면 모델
    │   ├── box.sdf                # 박스 모델
    │   └── sphere.sdf             # 구체 모델
    └── world.sdf                  # 3장 월드 파일
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

# 예: 2장 튜토리얼의 객체가 포함된 월드 실행
cd ../chapter2
ign gazebo -r world_with_objects.sdf
```