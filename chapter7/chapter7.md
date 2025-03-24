# 7장: 모듈식 로봇 모델링 - 센서와 액추에이터 체계화

이 챕터에서는 이전 장에서 만든 차동 구동 로봇을 더 체계적으로 구성하는 방법을 알아봅니다. 모델의 디렉토리 구조를 센서(sensors)와 액추에이터(actuators)로 분류하여 모듈식 설계를 구현하는 방법을 배웁니다.

## 1. 모듈식 로봇 설계의 장점

모듈식 설계는 다음과 같은 장점을 제공합니다:

1. **재사용성**: 한 번 만든 센서나 액추에이터 모듈을 여러 로봇에서 재사용할 수 있습니다.
2. **유지 보수**: 개별 모듈을 독립적으로 수정하거나 업그레이드할 수 있습니다.
3. **확장성**: 새로운 센서나 액추에이터를 쉽게 추가할 수 있습니다.
4. **명확한 구조**: 로봇의 구성 요소를 기능별로 분류하여 이해하기 쉽습니다.

## 2. 디렉토리 구조

체계화된 디렉토리 구조는 다음과 같습니다:

```
chapter7/
├── chapter7.md
├── diff_drive_world.sdf
└── models/
    ├── diff_drive_robot/
    │   ├── model.config
    │   ├── model.sdf       # 메인 로봇 모델 (센서, 액추에이터 참조 및 차동 구동 플러그인 포함)
    │   ├── actuators/      # 액추에이터 디렉토리
    │   │   ├── left_wheel/
    │   │   │   ├── model.config
    │   │   │   └── model.sdf   # 좌측 바퀴 모델
    │   │   └── right_wheel/
    │   │       ├── model.config
    │   │       └── model.sdf   # 우측 바퀴 모델
    │   └── sensors/        # 센서 디렉토리
    │       └── imu/
    │           ├── model.config
    │           └── model.sdf   # IMU 센서 모델
    └── ground_plane/
        ├── model.config
        └── model.sdf
```

## 3. 모듈 분리하기

### 3.1 센서 모듈화 (IMU 예시)

IMU 센서를 독립적인 모듈로 분리하여 `sensors/imu/` 디렉토리에 배치합니다:

```xml
<!-- sensors/imu/model.sdf -->
<?xml version="1.0"?>
<sdf version="1.6">
  <model name="imu_sensor">
    <link name="imu_link">
      <pose>0 0 0 0 0 0</pose>
      <inertial>
        <mass>0.1</mass>
        <inertia>
          <ixx>0.000001</ixx>
          <iyy>0.000001</iyy>
          <izz>0.000001</izz>
          <ixy>0</ixy>
          <ixz>0</ixz>
          <iyz>0</iyz>
        </inertia>
      </inertial>
      <collision name="collision">
        <geometry>
          <box>
            <size>0.05 0.05 0.02</size>
          </box>
        </geometry>
      </collision>
      <visual name="visual">
        <geometry>
          <box>
            <size>0.05 0.05 0.02</size>
          </box>
        </geometry>
        <material>
          <ambient>0.8 0.2 0.2 1</ambient>
          <diffuse>0.8 0.2 0.2 1</diffuse>
          <specular>0.1 0.1 0.1 1</specular>
        </material>
      </visual>
      <sensor name="imu_sensor" type="imu">
        <always_on>1</always_on>
        <update_rate>100</update_rate>
        <visualize>true</visualize>
        <topic>imu</topic>
        <plugin filename="libignition-gazebo-imu-system.so"
                name="ignition::gazebo::systems::Imu">
        </plugin>
      </sensor>
    </link>
  </model>
</sdf>
```

### 3.2 액추에이터 모듈화

#### 3.2.1 바퀴 모듈

바퀴를 재사용 가능한 별도의 좌/우 바퀴 모듈로 분리합니다:

```xml
<!-- actuators/left_wheel/model.sdf -->
<?xml version="1.0"?>
<sdf version="1.6">
  <model name="left_wheel">
    <link name="left_wheel_link">
      <pose>0 0 0 0 0 0</pose>
      <inertial>
        <mass>1.0</mass>
        <inertia>
          <ixx>0.0029</ixx>
          <iyy>0.0029</iyy>
          <izz>0.0052</izz>
          <ixy>0</ixy>
          <ixz>0</ixz>
          <iyz>0</iyz>
        </inertia>
      </inertial>
      <collision name="collision">
        <geometry>
          <cylinder>
            <radius>0.05</radius>
            <length>0.04</length>
          </cylinder>
        </geometry>
      </collision>
      <visual name="visual">
        <geometry>
          <cylinder>
            <radius>0.05</radius>
            <length>0.04</length>
          </cylinder>
        </geometry>
        <material>
          <ambient>0.1 0.1 0.1 1</ambient>
          <diffuse>0.1 0.1 0.1 1</diffuse>
          <specular>0.1 0.1 0.1 1</specular>
        </material>
      </visual>
    </link>
  </model>
</sdf>
```

```xml
<!-- actuators/right_wheel/model.sdf -->
<?xml version="1.0"?>
<sdf version="1.6">
  <model name="right_wheel">
    <link name="right_wheel_link">
      <pose>0 0 0 0 0 0</pose>
      <inertial>
        <mass>1.0</mass>
        <inertia>
          <ixx>0.0029</ixx>
          <iyy>0.0029</iyy>
          <izz>0.0052</izz>
          <ixy>0</ixy>
          <ixz>0</ixz>
          <iyz>0</iyz>
        </inertia>
      </inertial>
      <collision name="collision">
        <geometry>
          <cylinder>
            <radius>0.05</radius>
            <length>0.04</length>
          </cylinder>
        </geometry>
      </collision>
      <visual name="visual">
        <geometry>
          <cylinder>
            <radius>0.05</radius>
            <length>0.04</length>
          </cylinder>
        </geometry>
        <material>
          <ambient>0.1 0.1 0.1 1</ambient>
          <diffuse>0.1 0.1 0.1 1</diffuse>
          <specular>0.1 0.1 0.1 1</specular>
        </material>
      </visual>
    </link>
  </model>
</sdf>
```

### 3.3 차동 구동 플러그인

차동 구동 제어를 위해 메인 로봇 모델에 다음 플러그인을 추가합니다:

```xml
<!-- 차동 구동 플러그인 -->
<plugin
  filename="libignition-gazebo-diff-drive-system.so"
  name="ignition::gazebo::systems::DiffDrive">
  <left_joint>left_wheel_joint</left_joint>
  <right_joint>right_wheel_joint</right_joint>
  <wheel_separation>0.35</wheel_separation>
  <wheel_radius>0.05</wheel_radius>
  <odom_publish_frequency>10</odom_publish_frequency>
  <topic>/cmd_vel</topic>
</plugin>
```

## 4. 메인 로봇 모델에서 모듈 조립하기

메인 로봇 모델에서는 이전에 분리한 모듈들을 참조하여 조립합니다:

```xml
<!-- 로봇 메인 모델 (model.sdf) -->
<?xml version="1.0"?>
<sdf version="1.6">
  <model name="diff_drive_robot">
    <!-- 로봇 본체 -->
    <link name="chassis">
      <!-- 본체 상세 정보 -->
    </link>

    <!-- 좌측 바퀴 모델 포함 -->
    <include>
      <uri>model://chapter7/models/diff_drive_robot/actuators/left_wheel</uri>
      <name>left_wheel</name>
      <pose>0.1 0.175 0.05 -1.5707 0 0</pose>
    </include>

    <!-- 우측 바퀴 모델 포함 -->
    <include>
      <uri>model://chapter7/models/diff_drive_robot/actuators/right_wheel</uri>
      <name>right_wheel</name>
      <pose>0.1 -0.175 0.05 -1.5707 0 0</pose>
    </include>

    <!-- 캐스터 휠 -->
    <link name="caster">
      <!-- 캐스터 상세 정보 -->
    </link>

    <!-- 조인트 정의 -->
    <joint name="left_wheel_joint" type="revolute">
      <parent>chassis</parent>
      <child>left_wheel::left_wheel_link</child>
      <!-- 축 상세 정보 -->
    </joint>

    <joint name="right_wheel_joint" type="revolute">
      <parent>chassis</parent>
      <child>right_wheel::right_wheel_link</child>
      <!-- 축 상세 정보 -->
    </joint>

    <joint name="caster_joint" type="ball">
      <parent>chassis</parent>
      <child>caster</child>
    </joint>

    <!-- IMU 센서 모델 포함 -->
    <include>
      <uri>model://chapter7/models/diff_drive_robot/sensors/imu</uri>
      <name>imu</name>
      <pose>0 0 0.1 0 0 0</pose>
    </include>

    <!-- IMU 센서 고정 조인트 -->
    <joint name="imu_joint" type="fixed">
      <parent>chassis</parent>
      <child>imu::imu_link</child>
    </joint>

    <!-- 차동 구동 플러그인 -->
    <plugin
      filename="libignition-gazebo-diff-drive-system.so"
      name="ignition::gazebo::systems::DiffDrive">
      <left_joint>left_wheel_joint</left_joint>
      <right_joint>right_wheel_joint</right_joint>
      <wheel_separation>0.35</wheel_separation>
      <wheel_radius>0.05</wheel_radius>
      <odom_publish_frequency>10</odom_publish_frequency>
      <topic>/cmd_vel</topic>
    </plugin>
  </model>
</sdf>
```

## 5. 모듈 참조 시 주의사항

1. **이름 지정**: 각 모듈 포함 시 고유한 이름을 지정해야 합니다. (예: `<name>left_wheel</name>`)
2. **참조 경로**: URI를 통해 모듈 경로를 정확히 지정해야 합니다. (예: `model://chapter7/models/diff_drive_robot/sensors/imu`)
3. **조인트 연결**: 모듈의 링크 참조 시 `모듈이름::링크이름` 형식을 사용합니다. (예: `left_wheel::left_wheel_link`)
4. **위치 지정**: 각 모듈의 상대적 위치를 `<pose>` 태그로 지정합니다.
5. **고유한 링크 이름**: 각 모듈 내의 링크에 고유한 이름을 부여하세요. (예: `left_wheel_link`, `right_wheel_link`). 일반적인 `link`라는 이름은 중복 에러의 원인이 됩니다.

## 6. 시뮬레이션 실행

다음 명령어로 모듈식 로봇 시뮬레이션을 실행합니다:

```bash
cd ~/git/ros2_gazebo_ignition_tutorial
export IGN_GAZEBO_RESOURCE_PATH=$PWD
ign gazebo -r chapter7/diff_drive_world.sdf
```

로봇을 Teleop으로 조종하여 움직일 수 있습니다.

### 6.1 IMU 데이터 확인하기

IMU 데이터를 확인하려면 다음 방법들을 사용할 수 있습니다:

1. **Topic Echo 플러그인 사용**:
   - Gazebo GUI에서 Topic Echo 플러그인을 열고 드롭다운 메뉴에서 `/imu` 토픽을 선택합니다.
   - 만약 Topic Echo 플러그인이 보이지 않는다면, 상단 메뉴의 플러그인 아이콘을 클릭하고 'Topic Echo'를 선택합니다.

2. **터미널에서 토픽 구독**:
   - 새 터미널을 열고 다음 명령어를 실행합니다:
   ```bash
   ign topic -e -t /imu
   ```

3. **로그 확인**:
   - IMU 데이터가 발행되고 있는지 확인하려면 다음 명령어로 토픽 목록을 확인합니다:
   ```bash
   ign topic -l | grep imu
   ```

로봇을 회전시키거나 이동시키면 IMU 센서가 측정한 값이 `/imu` 토픽으로 발행됩니다. 특히 회전시키면 각속도 값이 크게 변화하는 것을 확인할 수 있습니다.

## 7. 모듈 추가하기

이 체계화된 구조를 활용하면 새로운 센서나 액추에이터를 쉽게 추가할 수 있습니다:

### 7.1 새로운 센서 추가 예시

LIDAR 센서를 추가하려면:

1. `sensors/lidar/` 디렉토리 생성
2. `model.config`와 `model.sdf` 파일 작성
3. 메인 로봇 모델에서 참조:
   ```xml
   <include>
     <uri>model://chapter7/models/diff_drive_robot/sensors/lidar</uri>
     <name>lidar</name>
     <pose>0.2 0 0.1 0 0 0</pose>
   </include>
   <joint name="lidar_joint" type="fixed">
     <parent>chassis</parent>
     <child>lidar::link</child>
   </joint>
   ```

### 7.2 새로운 액추에이터 추가 예시

로봇 팔을 추가하려면:

1. `actuators/arm/` 디렉토리 생성
2. `model.config`와 `model.sdf` 파일 작성
3. 메인 로봇 모델에서 참조:
   ```xml
   <include>
     <uri>model://chapter7/models/diff_drive_robot/actuators/arm</uri>
     <name>robot_arm</name>
     <pose>0.2 0 0.1 0 0 0</pose>
   </include>
   <joint name="arm_joint" type="fixed">
     <parent>chassis</parent>
     <child>robot_arm::base_link</child>
   </joint>
   ```

## 8. 모듈식 설계의 확장성

이러한 모듈식 접근 방식은 다음과 같은 확장성을 제공합니다:

1. **센서 라이브러리 구축**: 다양한 센서 모듈을 만들어 라이브러리화할 수 있습니다.
2. **로봇 구성 변경**: 모듈을 추가하거나 제거하여 다양한 로봇 구성을 만들 수 있습니다.
3. **코드 재사용**: 잘 설계된 모듈은 다른 프로젝트에서도 사용할 수 있습니다.
4. **팀 협업**: 팀원들이 서로 다른 모듈을 개발하여 통합할 수 있습니다.

## 9. 문제 해결

- **모듈 참조 오류**: URI 경로가 올바른지 확인하세요.
- **조인트 연결 문제**: 자식 링크 이름이 `모듈이름::링크이름` 형식으로 정확히 지정되었는지 확인하세요.
- **충돌 감지 문제**: 모듈 간의 위치가 겹치지 않는지 확인하세요.
- **링크 이름 중복 문제**: 
  - 동일한 모듈을 여러 번 포함할 경우(예: 바퀴 2개) 링크 이름 중복으로 인해 Gazebo가 자동으로 `wheel_link(1)` 형태로 이름을 변경할 수 있습니다.
  - 해결책 1: 각 모듈 안에서 링크 이름을 모듈 역할에 따라 고유하게 설정합니다(예: `left_wheel_link`, `right_wheel_link`).
  - 해결책 2: 모듈을 참조할 때 Gazebo가 변경한 이름을 사용합니다(예: `right_wheel::wheel_link(1)`).
  - 실행 로그를 확인하여 Gazebo가 링크 이름을 어떻게 변경했는지 확인할 수 있습니다.
- **IMU 데이터가 나타나지 않는 문제**:
  - 센서 토픽 이름을 확인하세요. Gazebo에서는 기본적으로 토픽 이름이 `/`로 시작해야 합니다. 
  - 예: `<topic>imu</topic>` 대신 `<topic>/imu</topic>`으로 설정
  - world 파일에 IMU 시스템 플러그인이 포함되어 있는지 확인하세요.
  - 다음 명령어로 토픽 발행 여부를 확인하세요: `ign topic -l | grep imu`
  - IMU 센서가 잘 보이는 위치에 있는지 확인하고, `<visualize>true</visualize>` 옵션이 설정되어 있는지 확인하세요.

## 다음 단계

이제 모듈식 로봇 모델링 방법을 배웠습니다. 다음 챕터에서는 이 모듈식 구조를 활용하여 더 복잡한 로봇을 만들고 ROS2와 통합하는 방법을 알아보겠습니다. 