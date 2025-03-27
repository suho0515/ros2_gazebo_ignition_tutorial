# 챕터 8: 기어비 실험과 바퀴 직접 제어

이 챕터에서는 로봇의 기어비 설정이 움직임에 미치는 영향을 실험하고 바퀴를 직접 제어하는 방법을 알아봅니다. 기어비는 모터의 회전 속도와 토크의 관계를 결정하는 중요한 요소로, 로봇의 속도와 힘에 직접적인 영향을 미칩니다.

## 8.1 기어비의 개념과 중요성

기어비는 입력 기어와 출력 기어 사이의 회전 속도 비율을 나타냅니다. 로봇 시스템에서 기어비는 다음과 같은 영향을 미칩니다:

- **속도와 토크의 교환**: 높은 기어비는 속도를 감소시키지만 토크를 증가시킵니다. 반대로 낮은 기어비는 속도는 증가하지만 토크는 감소합니다.
- **정밀 제어**: 기어비는 모터 회전의 정밀도에도 영향을 미칩니다.
- **에너지 효율**: 적절한 기어비 선택은 로봇의 에너지 효율을 최적화하는 데 중요합니다.

Gazebo 시뮬레이터에서 기어비는 `joint` 요소의 `physics` 섹션에서 `gear_ratio` 매개변수로 설정할 수 있습니다.

## 8.2 기어비가 다른 두 로봇 모델 설계

이 실험을 위해 두 개의 다른 차동 구동 로봇 모델을 만들 것입니다:

1. **낮은 기어비 로봇(1:1)**: 빠른 속도를 가지지만 상대적으로 낮은 토크를 가진 로봇
2. **높은 기어비 로봇(3:1)**: 느린 속도지만 높은 토크를 가진 로봇

### 8.2.1 낮은 기어비 모델 (1:1)

낮은 기어비 모델은 `gear_ratio`를 1로 설정하여 모터의 입력 속도가 1:1 비율로 바퀴에 직접 전달되도록 합니다.

```xml
<!-- 좌측 바퀴 조인트 - 기어비 1:1 (낮은 기어비) -->
<joint name="left_wheel_joint" type="revolute">
  <parent>chassis</parent>
  <child>left_wheel::left_wheel_link</child>
  <axis>
    <xyz>0 0 1</xyz>
    <!-- 기타 설정 -->
  </axis>
  <physics>
    <ode>
      <gear_ratio>1</gear_ratio>
    </ode>
  </physics>
</joint>
```

### 8.2.2 높은 기어비 모델 (3:1)

높은 기어비 모델은 `gear_ratio`를 3으로 설정하여 모터가 3번 회전할 때 바퀴가 1번 회전하도록 합니다. 이로 인해 로봇은 더 느리게 움직이지만 더 높은 토크(힘)를 갖게 됩니다.

```xml
<!-- 좌측 바퀴 조인트 - 기어비 3:1 (높은 기어비) -->
<joint name="left_wheel_joint" type="revolute">
  <parent>chassis</parent>
  <child>left_wheel::left_wheel_link</child>
  <axis>
    <xyz>0 0 1</xyz>
    <!-- 기타 설정 -->
  </axis>
  <physics>
    <ode>
      <gear_ratio>3</gear_ratio>
    </ode>
  </physics>
</joint>
```

## 8.3 바퀴 직접 제어를 위한 JointController 플러그인

기존의 DiffDrive 플러그인은 로봇의 선속도와 각속도를 제어하는 상위 수준의 제어 메커니즘입니다. 그러나 기어비의 효과를 정확하게 관찰하려면 각 바퀴의 회전 속도를 직접 제어해야 합니다.

이를 위해 JointController 플러그인을 사용하여 각 바퀴 조인트에 직접 속도 명령을 보낼 수 있습니다.

```xml
<!-- 좌측 바퀴 제어를 위한 JointController 플러그인 -->
<plugin
  filename="libignition-gazebo-joint-controller-system.so"
  name="ignition::gazebo::systems::JointController">
  <joint_name>left_wheel_joint</joint_name>
  <topic>/wheel_velocity</topic>
  <joint_type>velocity</joint_type>
  <p_gain>1</p_gain>
  <i_gain>0.1</i_gain>
  <d_gain>0.01</d_gain>
</plugin>
```

위 설정에서 `topic`은 바퀴 회전 속도를 제어하는 데 사용되는 메시지 토픽을 지정합니다. 두 로봇의 모든 바퀴는 동일한 토픽을 통해 속도 명령을 받기 때문에 동일한 입력을 받아도 기어비에 따라 다르게 반응할 것입니다.

## 8.4 GUI로 바퀴 속도 제어하기

Gazebo GUI에서 바퀴 속도를 쉽게 제어할 수 있도록 Publisher 플러그인을 추가합니다:

```xml
<!-- 바퀴 속도 제어를 위한 Publisher 플러그인 -->
<plugin
  filename="libignition-gazebo-gui-system.so"
  name="ignition::gazebo::systems::Gui">
  <plugin
    filename="libignition-gazebo-publisher-system.so"
    name="ignition::gazebo::systems::Publisher">
    <topic>/wheel_velocity</topic>
    <type>ignition.msgs.Double</type>
    <value>0.0</value>
    <description>바퀴 속도 제어</description>
    <min>-10</min>
    <max>10</max>
    <step>0.1</step>
  </plugin>
</plugin>
```

이 GUI 슬라이더를 사용하여 -10에서 10 사이의 속도 값(라디안/초)을 설정할 수 있으며, 이 값은 양쪽 바퀴 모두에 적용됩니다.

## 8.5 실험 실행 및 관찰

실험을 실행하려면 다음 명령어를 사용합니다:

```bash
cd ~/git/ros2_gazebo_ignition_tutorial
export IGN_GAZEBO_RESOURCE_PATH=$PWD
ign gazebo -r chapter8/diff_drive_world.sdf
```

시뮬레이션이 시작되면 다음 사항을 관찰할 수 있습니다:

1. GUI 슬라이더를 사용하여 바퀴 속도를 조절합니다.
2. 동일한 속도 명령에 대해 두 로봇이 얼마나 다르게 움직이는지 관찰합니다:
   - 낮은 기어비(1:1) 로봇(녹색): 빠르게 움직이지만 가속과 감속이 빠릅니다.
   - 높은 기어비(3:1) 로봇(빨간색): 느리게 움직이지만 더 강한 힘과 안정된 움직임을 보입니다.

## 8.6 터미널에서 바퀴 속도 제어하기

GUI 외에도 터미널에서 직접 명령어를 사용하여 바퀴 속도를 제어할 수 있습니다:

```bash
ign topic -t /wheel_velocity -m ignition.msgs.Double -p "data: 5.0"
```

이 명령은 바퀴에 5.0 라디안/초의 속도를 설정합니다. 양수 값은 앞으로 이동, 음수 값은 뒤로 이동을 의미합니다.

## 8.7 결론 및 응용

이 실험을 통해 기어비가 로봇의 움직임 특성에 어떤 영향을 미치는지 확인했습니다. 실제 로봇 설계 시 고려해야 할 중요한 포인트는 다음과 같습니다:

- 빠른 속도와 민첩성이 필요한 경우: 낮은 기어비 선택
- 강한 토크와 안정성이 필요한 경우: 높은 기어비 선택
- 특정 작업에 맞는 최적의 기어비 조합 찾기

다음 챕터에서는 로봇 제어를 위한 ROS 2 인터페이스를 구축하고, 더 복잡한 제어 알고리즘을 개발하는 방법을 알아보겠습니다. 