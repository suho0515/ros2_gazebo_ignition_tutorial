# 5장: Gazebo GUI에서 Teleop 플러그인으로 로봇 조작

이 챕터에서는 Gazebo GUI에서 Teleop 플러그인을 사용하여 로봇을 조작하는 방법을 알아봅니다. Teleop 플러그인은 직관적인 GUI를 통해 로봇을 쉽게 제어할 수 있게 해줍니다.

## 1. 디렉토리 구조

chapter5의 디렉토리 구조는 chapter4와 유사합니다:

```
chapter5/
├── chapter5.md
├── diff_drive_world.sdf
└── models/
    ├── diff_drive_robot/
    │   ├── model.config
    │   └── model.sdf
    └── ground_plane/
        ├── model.config
        └── model.sdf
```

## 2. Teleop GUI 플러그인 설정

`diff_drive_world.sdf` 파일의 GUI 섹션에 Teleop 플러그인을 추가합니다:

```xml
<plugin filename="Teleop" name="Teleop">
  <gz-gui>
    <property key="showTitleBar" type="bool">false</property>
    <property key="resizable" type="bool">true</property>
    <property key="width" type="double">400</property>
    <property key="height" type="double">400</property>
    <property key="state" type="string">docked</property>
  </gz-gui>
  <topic>/cmd_vel</topic>
  <max_linear>1.0</max_linear>
  <max_angular>0.5</max_angular>
</plugin>
```

이 플러그인은 다음과 같은 기능을 제공합니다:
- 방향키 버튼을 통한 로봇 제어
- 최대 속도 설정
- 키보드 입력 또는 슬라이더를 통한 제어 방식 선택

## 3. 차동 구동 플러그인 설정

로봇 모델에는 차동 구동 플러그인을 추가하여 GUI로 제어할 수 있도록 합니다:

```xml
<plugin filename="libignition-gazebo-diff-drive-system.so"
        name="ignition::gazebo::systems::DiffDrive">
  <left_joint>left_wheel_joint</left_joint>
  <right_joint>right_wheel_joint</right_joint>
  <wheel_separation>0.35</wheel_separation>
  <wheel_radius>0.05</wheel_radius>
  <odom_publish_frequency>10</odom_publish_frequency>
  <topic>cmd_vel</topic>
</plugin>
```

## 4. 키보드 제어를 위한 TriggeredPublisher 설정

키보드로 직접 명령을 보낼 수 있도록 TriggeredPublisher 플러그인도 추가되어 있습니다:

```xml
<!-- 전진 (위 화살표) -->
<plugin filename="libignition-gazebo-triggered-publisher-system.so"
        name="ignition::gazebo::systems::TriggeredPublisher">
    <input type="ignition.msgs.Int32" topic="/keyboard/keypress">
        <match field="data">16777235</match>
    </input>
    <output type="ignition.msgs.Twist" topic="/cmd_vel">
        linear: {x: 0.5}, angular: {z: 0.0}
    </output>
</plugin>
```

이러한 플러그인을 통해 3D 뷰를 클릭한 후 키보드의 화살표 키로도 로봇을 제어할 수 있습니다.

## 5. 모델 경로 설정

Gazebo Fortress에서는 모델 경로를 올바르게 설정해야 합니다:

```xml
<include>
  <uri>model://chapter5/models/ground_plane</uri>
</include>

<include>
  <uri>model://chapter5/models/diff_drive_robot</uri>
  <pose>0 0 0 0 0 0</pose>
</include>
```

## 6. 시뮬레이션 실행 및 로봇 조작

### 6.1 시뮬레이션 실행

시뮬레이션을 시작하기 전에 모델 경로를 설정합니다:

```bash
cd ~/git/ros2_gazebo_ignition_tutorial
export IGN_GAZEBO_RESOURCE_PATH=$PWD
ign gazebo -r chapter5/diff_drive_world.sdf
```

### 6.2 로봇 조작

Gazebo GUI가 시작되면 Teleop 패널을 사용하여 로봇을 제어할 수 있습니다:

1. 오른쪽에 나타나는 Teleop 패널에서 다음 방법으로 로봇을 조작할 수 있습니다:
   - **BUTTONS 탭**: 방향 버튼을 클릭하여 로봇 이동
   - **KEYBOARD 탭**: 방향키 아이콘을 클릭하여 로봇 이동
   - **SLIDERS 탭**: 슬라이더를 조정하여 로봇의 속도와 방향 제어

2. 직접 3D 뷰를 클릭한 후 키보드 화살표 키를 사용하여 로봇을 제어할 수도 있습니다:
   - **↑**: 전진
   - **↓**: 후진
   - **←**: 좌회전
   - **→**: 우회전
   - **Space**: 정지

3. Topic과 최대 속도를 Teleop 패널에서 직접 조정할 수도 있습니다.

## 7. 문제 해결

- **모델을 찾을 수 없음**: 모델 경로가 올바르게 설정되었는지 확인하세요. `IGN_GAZEBO_RESOURCE_PATH` 환경 변수에 현재 작업 디렉토리가 포함되어 있어야 합니다.
- **Teleop 패널이 나타나지 않음**: GUI 우측 상단의 플러그인 메뉴(수직 점 세 개)를 클릭하고 Teleop을 선택하세요.
- **로봇이 움직이지 않음**: 로봇의 차동 구동 플러그인이 올바르게 설정되었는지, 그리고 Topic 이름(`/cmd_vel`)이 일치하는지 확인하세요.

## 다음 단계

이제 Gazebo GUI에서 Teleop 플러그인을 사용하여 로봇을 조작하는 방법을 배웠습니다. 다음 챕터에서는 로봇에 센서를 추가하고, ROS2와의 통합을 통해 더 복잡한 로봇 애플리케이션을 개발하는 방법을 알아보겠습니다. 