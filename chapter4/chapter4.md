# 4장: 간단한 이동 로봇 모델 생성 및 구동

이 챕터에서는 Gazebo Ignition을 사용하여 좌우 구동 바퀴 2개와 캐스터 1개로 구성된 간단한 이동 로봇을 생성하고 시뮬레이션하는 방법을 알아봅니다. 또한 효율적인 디렉토리 구조를 통해 모델을 체계적으로 관리하는 방법도 배웁니다.

## 1. 체계적인 디렉토리 구조

이동 로봇을 위한 효율적인 디렉토리 구조는 다음과 같습니다:

```
chapter4/
├── chapter4.md
├── diff_drive_world.sdf
└── models/
    ├── diff_drive_robot/
    │   ├── model.config
    │   └── model.sdf
    └── ground_plane/
        ├── model.config
        └── model.sdf
```

이 구조의 특징:
- **models 디렉토리**: 각 모델은 독립적인 하위 디렉토리를 가집니다.
- **model.config**: Gazebo에서 모델을 인식하는 데 필요한 메타데이터를 담습니다.

## 2. 이동 로봇 모델 설계

간단한 이동 로봇의 주요 구성 요소:

1. **본체(Chassis)**: 로봇의 기본 프레임
   - 크기: 가로 0.4m, 세로 0.3m, 높이 0.1m
   - 무게: 5.0kg
   - 색상: 파란색

2. **좌/우 구동 바퀴**: 독립적으로 회전하여 로봇의 방향 제어
   - 직경: 0.1m (반지름 0.05m)
   - 폭: 0.04m
   - 무게: 각 1.0kg
   - 위치: 로봇 중심에서 좌우 0.175m 떨어진 위치
   - 색상: 검은색

3. **캐스터 휠**: 로봇의 안정성을 위한 피동 바퀴
   - 직경: 0.05m (반지름 0.025m)
   - 무게: 0.5kg
   - 위치: 로봇 앞쪽에서 0.15m 떨어진 중앙
   - 마찰: 매우 낮게 설정 (μ=0.1)
   - 색상: 회색

## 3. 모델 파일 구성

### 3.1 model.config 파일

`models/diff_drive_robot/model.config` 파일은 모델의 기본 정보를 정의합니다. 이 파일은 모델 검색과 식별에 사용됩니다.

```xml
<?xml version="1.0"?>
<model>
  <name>diff_drive_robot</name>
  <version>1.0</version>
  <sdf version="1.6">model.sdf</sdf>

  <author>
    <name>사용자 이름</name>
    <email>your.email@example.com</email>
  </author>

  <description>
    간단한 차동 구동 로봇 모델 (2개의 구동 바퀴와 1개의 캐스터)
  </description>
</model>
```

### 3.2 model.sdf 파일

`models/diff_drive_robot/model.sdf` 파일은 로봇의 실제 물리적 구조와 동작을 정의합니다. 이 파일은 다음과 같은 주요 요소로 구성됩니다:

1. **로봇 본체 정의**: 직육면체 형태로 정의
2. **좌측 바퀴 정의**: 원통형으로 정의
3. **우측 바퀴 정의**: 원통형으로 정의
4. **캐스터 휠 정의**: 구형으로 정의
5. **조인트 설정**: 바퀴와 본체를 연결하는 회전 조인트
6. **플러그인 설정**: 차동 구동 제어를 위한 플러그인

이 모델 구성을 통해 현실적인 차동 구동 로봇의 움직임을 시뮬레이션할 수 있습니다.

## 4. 월드 파일 구성

`diff_drive_world.sdf` 파일은 시뮬레이션 환경을 정의합니다. 이 파일은 다음 요소를 포함합니다:

1. **물리 설정**: 시뮬레이션의 시간 단계, 실시간 계수 등을 정의
2. **기본 플러그인**: Physics, UserCommands, SceneBroadcaster 등 기본 플러그인 포함
3. **조명 설정**: 태양광과 같은 방향성 조명 정의
4. **지면 모델**: 로봇이 이동할 평면 포함
5. **로봇 모델**: 차동 구동 로봇 모델 포함

월드 파일은 모델과 환경을 통합하여 완전한 시뮬레이션 장면을 구성합니다.

## 5. 차동 구동 로봇의 운동학

차동 구동(Differential Drive) 방식은 두 바퀴의 속도 차이를 이용해 로봇의 방향을 제어하는 메커니즘입니다.

### 5.1 기본 동작 원리

1. **직진 이동**: 양쪽 바퀴를 같은 속도로 회전
2. **회전**: 양쪽 바퀴를 다른 속도로 회전
   - 제자리 회전: 바퀴를 반대 방향으로 같은 속도로 회전
   - 원호 이동: 한쪽 바퀴를 더 빠르게 회전

### 5.2 운동학 방정식

로봇의 선속도(v)와 각속도(ω)는 바퀴 속도로부터 다음과 같이 계산됩니다:

- v = (v_right + v_left) / 2
- ω = (v_right - v_left) / L

여기서 v_right와 v_left는 각각 오른쪽과 왼쪽 바퀴의 선속도이고, L은 바퀴 간 거리입니다.

반대로, 원하는 선속도와 각속도를 바퀴 속도로 변환하는 방정식은 다음과 같습니다:

- v_left = v - ω * L/2
- v_right = v + ω * L/2

이 방정식을 사용하여 로봇의 움직임을 제어할 수 있습니다.

## 6. 시뮬레이션 실행 및 로봇 제어

### 6.1 시뮬레이션 실행

다음 명령어로 시뮬레이션을 시작합니다:

```bash
cd chapter4
ign gazebo -r worlds/diff_drive_world.sdf
```

### 6.2 로봇 제어

터미널에서 다음 명령어를 사용하여 로봇을 제어할 수 있습니다:

```bash
# 직진 이동 (선속도 0.5 m/s)
ign topic -t "/cmd_vel" -m ignition.msgs.Twist -p "linear: {x: 0.5}, angular: {z: 0.0}"

# 제자리 회전 (각속도 0.5 rad/s)
ign topic -t "/cmd_vel" -m ignition.msgs.Twist -p "linear: {x: 0.0}, angular: {z: 0.5}"

# 원호 이동 (선속도 0.3 m/s, 각속도 0.2 rad/s)
ign topic -t "/cmd_vel" -m ignition.msgs.Twist -p "linear: {x: 0.3}, angular: {z: 0.2}"

# 정지
ign topic -t "/cmd_vel" -m ignition.msgs.Twist -p "linear: {x: 0.0}, angular: {z: 0.0}"
```

## 7. 모델 확장 및 개선 방법

기본 모델을 더 발전시키는 방법:

### 7.1 센서 추가

로봇에 다양한 센서를 추가하여 기능을 확장할 수 있습니다:

1. **카메라**: 시각 정보 수집
   ```xml
   <link name="camera_link">
     <pose>0.2 0 0.1 0 0 0</pose>
     <sensor name="camera" type="camera">
       <camera>
         <horizontal_fov>1.047</horizontal_fov>
         <image>
           <width>640</width>
           <height>480</height>
         </image>
       </camera>
       <always_on>1</always_on>
       <update_rate>30</update_rate>
     </sensor>
   </link>
   ```

2. **라이다(LiDAR)**: 주변 환경 감지
   ```xml
   <link name="lidar_link">
     <pose>0 0 0.15 0 0 0</pose>
     <sensor name="lidar" type="gpu_lidar">
       <lidar>
         <scan>
           <horizontal>
             <samples>360</samples>
             <min_angle>-3.14159</min_angle>
             <max_angle>3.14159</max_angle>
           </horizontal>
         </scan>
         <range>
           <min>0.08</min>
           <max>10.0</max>
         </range>
       </lidar>
       <always_on>1</always_on>
       <update_rate>10</update_rate>
     </sensor>
   </link>
   ```

3. **IMU(관성 측정 장치)**: 로봇의 방향 및 가속도 측정
   ```xml
   <link name="imu_link">
     <pose>0 0 0.1 0 0 0</pose>
     <sensor name="imu_sensor" type="imu">
       <always_on>1</always_on>
       <update_rate>100</update_rate>
     </sensor>
   </link>
   ```

### 7.2 외관 개선

1. **메시 파일 사용**:
   ```xml
   <visual name="visual">
     <geometry>
       <mesh>
         <uri>model://diff_drive_robot/meshes/chassis.dae</uri>
         <scale>1 1 1</scale>
       </mesh>
     </geometry>
   </visual>
   ```

2. **텍스처 적용**:
   ```xml
   <material>
     <script>
       <uri>file://media/materials/scripts/gazebo.material</uri>
       <name>Gazebo/Wood</name>
     </script>
   </material>
   ```

## 8. 체계적인 모델 관리를 위한 추가 팁

### 8.1 모델 경로 설정

Gazebo가 모델을 찾을 수 있도록 경로를 설정합니다:

```bash
export IGN_GAZEBO_RESOURCE_PATH=$IGN_GAZEBO_RESOURCE_PATH:$PWD
```

### 8.2 디렉토리 구조 최적화

1. **공통 구성 요소 분리**: 자주 사용되는 바퀴, 센서 등은 별도 모델로 관리
2. **가독성 있는 파일 구조**: 관련 파일을 논리적으로 그룹화
3. **버전 관리**: 모델 변경 내역을 체계적으로 관리

### 8.3 문서화

각 모델과 구성 요소에 대한 명확한 문서화는 모델 재사용과 협업에 중요합니다. README 파일을 추가하고, 코드 내 주석을 충분히 제공하는 것이 좋습니다.

## 다음 단계

이제 간단한 차동 구동 로봇 모델을 생성하고 시뮬레이션하는 방법을 배웠습니다. 다음 챕터에서는 이 로봇에 센서를 추가하고 ROS2와 통합하여 더 복잡한 로봇 애플리케이션을 개발하는 방법을 알아보겠습니다.
