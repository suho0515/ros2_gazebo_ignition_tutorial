# 6장: 로봇에 IMU 센서 추가하기

이 챕터에서는 차동 구동 로봇에 IMU(관성 측정 장치) 센서를 추가하고, 센서 데이터를 시각화하는 방법을 알아봅니다. IMU 센서는 로봇의 방향, 가속도, 각속도 등을 측정하여 로봇의 움직임과 자세에 대한 중요한 정보를 제공합니다.

## 1. 디렉토리 구조

chapter6의 디렉토리 구조는 chapter5와 유사하지만, 로봇 모델에 IMU 센서가 추가됩니다:

```
chapter6/
├── chapter6.md
├── diff_drive_world.sdf
└── models/
    ├── diff_drive_robot/
    │   ├── model.config
    │   └── model.sdf       # IMU 센서가 추가된 로봇 모델
    └── ground_plane/
        ├── model.config
        └── model.sdf
```

## 2. IMU 센서 추가하기

### 2.1 IMU 링크 및 센서 설정

로봇의 `model.sdf` 파일에 IMU 센서를 추가합니다. IMU 센서는 별도의 링크로 정의하고, 로봇의 본체(chassis)에 고정됩니다:

```xml
<!-- IMU 센서 링크 추가 -->
<link name="imu_link">
  <pose>0 0 0.05 0 0 0</pose>
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

<!-- IMU 고정 조인트 -->
<joint name="imu_joint" type="fixed">
  <parent>chassis</parent>
  <child>imu_link</child>
</joint>
```

여기서 중요한 설정은 다음과 같습니다:
- `<always_on>1</always_on>`: 센서가 항상 활성화된 상태를 유지합니다.
- `<update_rate>100</update_rate>`: 센서가 초당 100번 데이터를 갱신합니다.
- `<visualize>true</visualize>`: 시뮬레이션에서 센서를 시각적으로 표시합니다.
- `<topic>imu</topic>`: 센서 데이터가 발행되는 토픽 이름을 지정합니다.

### 2.2 월드 파일에 IMU 시스템 플러그인 추가

시뮬레이션 월드 파일에 IMU 시스템 플러그인을 추가합니다:

```xml
<!-- IMU 시스템 플러그인 추가 -->
<plugin
  filename="libignition-gazebo-imu-system.so"
  name="ignition::gazebo::systems::Imu">
</plugin>
```

## 3. IMU 데이터 시각화

### 3.1 Plot 플러그인 설정

`diff_drive_world.sdf` 파일의 GUI 섹션에 IMU 데이터를 시각화하기 위한 Plot 플러그인을 추가합니다:

```xml
<!-- IMU 플롯 플러그인 추가 -->
<plugin filename="Plot" name="IMU 데이터">
  <gz-gui>
    <property key="showTitleBar" type="bool">true</property>
    <property key="resizable" type="bool">true</property>
    <property key="width" type="double">400</property>
    <property key="height" type="double">300</property>
    <property key="state" type="string">docked</property>
  </gz-gui>
  <topic>/imu</topic>
  <label>IMU</label>
  <maximum_points>100</maximum_points>
  <minimum_y>-10</minimum_y>
  <maximum_y>10</maximum_y>
  <measure>
    <type>angular_velocity</type>
    <label>Angular velocity</label>
    <dimension>
      <type>x</type>
      <color>0 0 255</color>
    </dimension>
    <dimension>
      <type>y</type>
      <color>255 0 0</color>
    </dimension>
    <dimension>
      <type>z</type>
      <color>0 255 0</color>
    </dimension>
  </measure>
</plugin>
```

이 플러그인은 `/imu` 토픽으로부터 데이터를 받아 각속도를 실시간으로 그래프로 시각화합니다. 
- x축 각속도: 파란색
- y축 각속도: 빨간색
- z축 각속도: 초록색

### 3.2 Topic Echo 플러그인 추가

IMU 데이터를 텍스트 형태로 확인하기 위해 Topic Echo 플러그인도 추가합니다:

```xml
<!-- 토픽 에코 플러그인 추가 -->
<plugin filename="TopicEcho" name="Topic Echo">
  <gz-gui>
    <property key="showTitleBar" type="bool">true</property>
    <property key="resizable" type="bool">true</property>
    <property key="width" type="double">400</property>
    <property key="height" type="double">300</property>
    <property key="state" type="string">docked</property>
  </gz-gui>
</plugin>
```

## 4. 시뮬레이션 실행 및 IMU 데이터 확인

### 4.1 시뮬레이션 실행

다음 명령어로 시뮬레이션을 시작합니다:

```bash
cd ~/git/ros2_gazebo_ignition_tutorial
export IGN_GAZEBO_RESOURCE_PATH=$PWD
ign gazebo -r chapter6/diff_drive_world.sdf
```

### 4.2 IMU 데이터 확인하기

1. **Plot 플러그인을 통한 시각화**:
   - Gazebo GUI에서 자동으로 표시되는 IMU 데이터 그래프를 통해 로봇의 각속도 변화를 실시간으로 확인할 수 있습니다.
   - 로봇을 회전시키면 특히 z축(yaw) 각속도에 변화가 생깁니다.

2. **Topic Echo를 사용한 데이터 확인**:
   - Topic Echo 플러그인을 통해 `/imu` 토픽을 선택하여 원시 데이터를 확인할 수 있습니다.
   - 이 플러그인을 사용하려면 GUI 우측 상단의 플러그인 메뉴(수직 점 세 개)를 클릭하고 Topic Echo를 선택한 후, 드롭다운에서 `/imu` 토픽을 선택합니다.

3. **터미널에서 확인하기**:
   - 별도의 터미널 창에서 다음 명령어를 실행하여 IMU 데이터를 확인할 수 있습니다:
   ```bash
   ign topic -e -t /imu
   ```

## 5. IMU 데이터 이해하기

IMU 센서는 다음과 같은 데이터를 제공합니다:

1. **선형 가속도 (Linear Acceleration)**:
   - 로봇의 x, y, z축 방향으로의 가속도
   - 단위: m/s²
   - 중력 가속도(약 9.8 m/s²)가 z축에 반영됨

2. **각속도 (Angular Velocity)**:
   - 로봇의 x, y, z축을 중심으로 한 회전 속도
   - 단위: rad/s
   - z축(yaw) 회전: 로봇이 좌/우로 회전할 때 변화
   - x축(roll) 회전: 로봇이 좌/우로 기울어질 때 변화
   - y축(pitch) 회전: 로봇이 앞/뒤로 기울어질 때 변화

3. **방향 (Orientation)**:
   - 쿼터니언(quaternion) 형태로 로봇의 3차원 방향 표현
   - 오일러 각도(roll, pitch, yaw)로 변환 가능

## 6. IMU 센서의 활용

IMU 센서는 다음과 같은 로봇 애플리케이션에서 중요하게 활용됩니다:

1. **로봇 자세 추정**: 로봇의 방향과 기울기를 정확하게 알 수 있습니다.
2. **이동 추정 (Odometry)**: 바퀴 엔코더와 함께 사용하여 로봇의 위치 추정 정확도를 높입니다.
3. **안정화 제어**: 로봇이 균형을 유지하거나 특정 자세를 유지하도록 제어합니다.
4. **장애물 감지**: 예상치 못한 충격이나 기울기 변화를 통해 장애물 감지에 활용할 수 있습니다.

## 7. 문제 해결

- **IMU 데이터가 표시되지 않음**: 
  - 모델 경로가 올바르게 설정되었는지 확인하세요.
  - IMU 시스템 플러그인이 월드 파일에 추가되었는지 확인하세요.
  - 센서의 토픽 이름이 `/imu`인지 확인하세요.

- **Plot 플러그인이 나타나지 않음**: 
  - GUI 우측 상단의 플러그인 메뉴에서 Plot을 선택하여 수동으로 추가하세요.
  - 새로운 Plot 인스턴스에서 `/imu` 토픽과 측정하고자 하는 데이터 타입을 선택하세요.

## 다음 단계

이제 로봇에 IMU 센서를 추가하고 데이터를 시각화하는 방법을 배웠습니다. 다음 챕터에서는 이 센서 데이터를 ROS2와 통합하여 더 복잡한 로봇 애플리케이션을 개발하는 방법을 알아보겠습니다. 