# 9장: 고급 로봇 모델 - 회전 가능한 드라이빙 모듈 설계

이 챕터에서는 로봇 모델링의 복잡도를 한 단계 높여, 회전 가능한 드라이빙 모듈을 갖춘 고급 로봇을 설계합니다. 이 로봇은 직사각형 베이스에 전후방에 각각 하나씩 장착된 드라이빙 모듈을 통해 전방향 이동성을 제공합니다.

## 1. 로봇 구조 개요

우리가 구현할 로봇의 구조는 다음과 같습니다:

1. **베이스 프레임**: 직사각형 형태의 본체
2. **드라이빙 모듈**: 
   - 원형 플랫폼에 좌우 바퀴가 장착된 구조
   - 모듈 자체가 좌우 90도(-90° ~ +90°)까지 회전 가능
   - 각 바퀴는 RPM 기반으로 독립적으로 제어 가능
3. **배치**: 
   - 전면에 하나, 후면에 하나의 드라이빙 모듈 장착
   - 각 모듈은 독립적으로 회전 및 제어 가능

이러한 구조는 일반적인 차동 구동 로봇보다 훨씬 높은 이동성을 제공합니다. 특히 협소한 공간에서의 방향 전환과 정밀한 위치 제어가 가능합니다.

## 2. 디렉토리 구조

```
chapter9/
├── chapter9.md
├── diff_drive_world.sdf
└── models/
    ├── advanced_robot/
    │   ├── model.config
    │   ├── model.sdf       # 메인 로봇 모델
    │   └── actuators/
    │       └── driving_module/
    │           ├── front/
    │           │   ├── model.config
    │           │   └── model.sdf   # 전면 드라이빙 모듈
    │           └── rear/
    │               ├── model.config
    │               └── model.sdf   # 후면 드라이빙 모듈
    └── ground_plane/
        ├── model.config
        └── model.sdf
```

## 3. 드라이빙 모듈 설계

각 드라이빙 모듈은 다음과 같은 특성을 가집니다:

### 3.1 모듈 베이스

드라이빙 모듈의 기본 구조는 원판 형태입니다:

```xml
<link name="module_base">
  <pose>0 0 0.05 0 0 0</pose>
  <inertial>
    <mass>2.0</mass>
    <inertia>
      <ixx>0.02</ixx>
      <iyy>0.02</iyy>
      <izz>0.04</izz>
      <!-- ... -->
    </inertia>
  </inertial>
  <collision name="collision">
    <geometry>
      <cylinder>
        <radius>0.15</radius>
        <length>0.05</length>
      </cylinder>
    </geometry>
  </collision>
  <visual name="visual">
    <!-- ... -->
  </visual>
</link>
```

### 3.2 좌우 바퀴

각 모듈에는 좌우에 바퀴가 장착되어 있습니다:

```xml
<link name="left_wheel">
  <pose>0 0.2 0.05 -1.5707 0 0</pose>
  <!-- ... -->
</link>

<link name="right_wheel">
  <pose>0 -0.2 0.05 -1.5707 0 0</pose>
  <!-- ... -->
</link>
```

### 3.3 바퀴 조인트 및 RPM 컨트롤러

각 바퀴는 모듈 베이스에 조인트로 연결되고, RPM 기반 컨트롤러를 통해 제어됩니다:

```xml
<joint name="left_wheel_joint" type="revolute">
  <parent>module_base</parent>
  <child>left_wheel</child>
  <axis>
    <xyz>0 0 1</xyz>
    <!-- ... -->
  </axis>
</joint>

<plugin
  filename="libignition-gazebo-joint-controller-system.so"
  name="ignition::gazebo::systems::JointController">
  <joint_name>left_wheel_joint</joint_name>
  <topic>/front_module/left_wheel/rpm</topic>
  <joint_type>velocity</joint_type>
  <!-- ... -->
  <transform_velocity_to_rpm>true</transform_velocity_to_rpm>
</plugin>
```

## 4. 메인 로봇 모델

메인 로봇 모델은 직사각형 베이스에 전후방에 드라이빙 모듈이 회전 조인트로 연결된 구조입니다:

### 4.1 베이스 프레임

```xml
<link name="base_frame">
  <pose>0 0 0.1 0 0 0</pose>
  <inertial>
    <mass>10.0</mass>
    <!-- ... -->
  </inertial>
  <collision name="collision">
    <geometry>
      <box>
        <size>0.8 0.5 0.1</size>
      </box>
    </geometry>
  </collision>
  <!-- ... -->
</link>
```

### 4.2 드라이빙 모듈 통합

전후방 드라이빙 모듈을 모델에 포함시킵니다:

```xml
<include>
  <uri>model://chapter9/models/advanced_robot/actuators/driving_module/front</uri>
  <name>front_module</name>
  <pose>0.35 0 0.1 0 0 0</pose>
</include>

<include>
  <uri>model://chapter9/models/advanced_robot/actuators/driving_module/rear</uri>
  <name>rear_module</name>
  <pose>-0.35 0 0.1 0 0 3.14159</pose>
</include>
```

### 4.3 회전 조인트 및 컨트롤러

각 드라이빙 모듈이 회전할 수 있도록 회전 조인트와 컨트롤러를 추가합니다:

```xml
<joint name="front_module_joint" type="revolute">
  <parent>base_frame</parent>
  <child>front_module::module_base</child>
  <axis>
    <xyz>0 0 1</xyz>
    <limit>
      <lower>-1.5708</lower>  <!-- -90도 (CCW 제한) -->
      <upper>1.5708</upper>   <!-- 90도 (CW 제한) -->
    </limit>
  </axis>
</joint>

<plugin
  filename="libignition-gazebo-joint-controller-system.so"
  name="ignition::gazebo::systems::JointController">
  <joint_name>front_module_joint</joint_name>
  <topic>/front_module/steering</topic>
  <joint_type>position</joint_type>
  <!-- ... -->
</plugin>
```

## 5. 월드 설정

월드 설정에서는 로봇 모델과 GUI 인터페이스를 정의합니다:

### 5.1 기본 설정

```xml
<world name="advanced_robot_world">
  <physics name="1ms" type="ode">
    <max_step_size>0.001</max_step_size>
    <real_time_factor>1.0</real_time_factor>
    <real_time_update_rate>1000</real_time_update_rate>
  </physics>

  <!-- 기본 플러그인 -->
  <plugin
    filename="libignition-gazebo-physics-system.so"
    name="ignition::gazebo::systems::Physics">
  </plugin>
  <!-- ... -->

  <!-- 지면 및 로봇 모델 포함 -->
  <include>
    <uri>model://chapter9/models/ground_plane</uri>
  </include>
  <include>
    <uri>model://chapter9/models/advanced_robot</uri>
    <pose>0 0 0 0 0 0</pose>
  </include>
</world>
```

### 5.2 GUI 인터페이스

각 드라이빙 모듈의 회전 각도와 바퀴 RPM을 제어할 수 있는 슬라이더를 포함한 GUI를 설정합니다:

```xml
<gui fullscreen="0">
  <!-- ... -->
  
  <!-- 전면 드라이빙 모듈 제어 -->
  <plugin filename="Publisher" name="Front Module Steering">
    <gz-gui>
      <property key="title" type="string">전면 모듈 회전 각도</property>
      <!-- ... -->
    </gz-gui>
    <topic>/front_module/steering</topic>
    <message_type>ignition.msgs.Double</message_type>
    <default_value>0.0</default_value>
    <min>-1.5708</min>
    <max>1.5708</max>
  </plugin>
  
  <plugin filename="Publisher" name="Front Left Wheel RPM">
    <!-- ... -->
    <topic>/front_module/left_wheel/rpm</topic>
    <!-- ... -->
  </plugin>
  
  <!-- 후면 드라이빙 모듈 제어 -->
  <!-- ... -->
</gui>
```

## 6. 시뮬레이션 실행 및 로봇 제어

완성된 로봇 모델로 시뮬레이션을 실행하고 제어해 보겠습니다:

### 6.1 시뮬레이션 실행

```bash
cd ~/git/ros2_gazebo_ignition_tutorial
export IGN_GAZEBO_RESOURCE_PATH=$PWD
ign gazebo -r chapter9/diff_drive_world.sdf
```

### 6.2 로봇 제어 방법

GUI에서 다음 컨트롤러를 사용하여 로봇을 제어할 수 있습니다:

1. **전면/후면 모듈 회전**: 각 모듈의 회전 각도 슬라이더를 조절하여 방향을 설정
2. **바퀴 RPM 제어**: 각 바퀴의 RPM 슬라이더를 조절하여 속도를 설정

다음은 몇 가지 기본 이동 패턴입니다:

- **직진 이동**: 
  - 전/후면 모듈 회전: 0도(중앙)
  - 전/후면 좌/우 바퀴: 동일한 양수 RPM(전진) 또는 음수 RPM(후진)

- **제자리 회전**: 
  - 전면 모듈: 90도(오른쪽) 또는 -90도(왼쪽)
  - 후면 모듈: -90도(오른쪽) 또는 90도(왼쪽) (전면과 반대 방향)
  - 모든 바퀴: 동일한 양수 RPM

- **측면 이동**:
  - 전/후면 모듈: 90도(오른쪽 이동) 또는 -90도(왼쪽 이동)
  - 전/후면 좌/우 바퀴: 동일한 RPM(양수는 오른쪽, 음수는 왼쪽)

### 6.3 터미널을 통한 명령 제어

GUI 외에도 터미널에서 직접 명령을 발행할 수 있습니다:

```bash
# 전면 모듈 45도 회전
ign topic -t /front_module/steering -m ignition.msgs.Double -p "data: 0.785"

# 후면 모듈 -45도 회전
ign topic -t /rear_module/steering -m ignition.msgs.Double -p "data: -0.785"

# 전면 좌측 바퀴 30 RPM 설정
ign topic -t /front_module/left_wheel/rpm -m ignition.msgs.Double -p "data: 30.0"
```

## 7. 제어 시나리오

이 로봇은 다양한 이동 패턴을 구현할 수 있습니다. 몇 가지 시나리오를 살펴보겠습니다:

### 7.1 전방향 이동 (Holonomic Movement)

고정된 방향을 유지하면서 어떤 방향으로든 이동 가능:

1. 전/후면 모듈을 동일한 방향으로 회전시킵니다 (예: 45도).
2. 모든 바퀴에 동일한 RPM을 적용합니다.
3. 로봇은 회전 없이 직선으로 움직입니다.

### 7.2 복잡한 경로 추종

아크 및 S자 경로를 따라 이동하기:

1. 전/후면 모듈의 각도를 다르게 설정합니다 (예: 전면 30도, 후면 -15도).
2. 각 모듈의 바퀴 RPM을 조절하여 원하는 경로를 따라갑니다.

### 7.3 제자리 회전과 동시에 이동

중심을 기준으로 회전하면서 이동하기:

1. 전면 모듈을 45도, 후면 모듈을 -45도로 설정합니다.
2. 전면 좌측 및 후면 우측: 양수 RPM
3. 전면 우측 및 후면 좌측: 음수 RPM

## 8. 문제 해결

- **모듈이 회전하지 않음**: 
  - 조인트 이름이 올바른지 확인하세요.
  - 회전 제한이 올바르게 설정되었는지 확인하세요.
  - PID 게인 값을 조정해 보세요.

- **바퀴가 회전하지 않음**: 
  - 각 바퀴의 토픽 이름이 올바른지 확인하세요.
  - RPM 값의 범위가 적절한지 확인하세요.

- **로봇이 예상대로 움직이지 않음**: 
  - 모듈 회전 각도와 바퀴 RPM 조합이 의도한 동작을 생성하는지 확인하세요.
  - 물리 엔진의 마찰 설정을 확인하세요.

## 9. 확장 가능성

이 모델은 다음과 같은 방향으로 확장할 수 있습니다:

1. **자동화된 이동 제어**: 특정 경로를 자동으로 따라가는 제어 알고리즘 개발
2. **센서 통합**: 라이다, 카메라 등을 추가하여 환경 인식 기능 구현
3. **ROS2 통합**: ROS2 인터페이스를 추가하여 고급 로봇 제어 시스템 구현
4. **맵핑 및 내비게이션**: SLAM 알고리즘을 통한 지도 생성 및 자율 주행 구현

## 10. 다음 단계

이제 복잡한 구조의 로봇 모델을 성공적으로 구현했습니다. 다음 장에서는 이 로봇에 센서를 추가하고 ROS2와 통합하여 더 강력한 로봇 시스템을 구축하는 방법을 배울 것입니다. 