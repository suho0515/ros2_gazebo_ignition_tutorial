# 8장: 모듈식 로봇 - RPM 기반 커스텀 컨트롤러 구현

이 챕터에서는 7장에서 개발한 모듈식 로봇의 제어 방식을 변경합니다. 기존의 차동 구동 컨트롤러 대신 좌우 바퀴 각각 RPM 값을 받아 구동하는 커스텀 컨트롤러를 구현합니다. 이를 통해 바퀴 회전 속도를 직접 제어하는 방법을 배웁니다.

## 1. RPM 기반 제어의 장점

RPM(Revolutions Per Minute) 기반 제어 방식은 다음과 같은 장점이 있습니다:

1. **정밀한 속도 제어**: 각 바퀴의 회전 속도를 정밀하게 제어할 수 있습니다.
2. **독립적인 제어**: 좌우 바퀴를 독립적으로 제어하여 더 복잡한 움직임을 구현할 수 있습니다.
3. **실제 로봇 시스템 유사성**: 많은 실제 로봇 시스템이 모터 RPM을 직접 제어하는 방식을 사용합니다.
4. **센서 데이터 연동**: 센서 데이터에 기반한 정교한 제어 알고리즘을 구현하기 용이합니다.

## 2. 디렉토리 구조

이전 장의 모듈식 구조를 유지하면서 제어 방식만 변경합니다:

```
chapter8/
├── chapter8.md
├── diff_drive_world.sdf
└── models/
    ├── diff_drive_robot/
    │   ├── model.config
    │   ├── model.sdf       # RPM 컨트롤러를 포함한 메인 로봇 모델
    │   ├── actuators/      
    │   │   ├── left_wheel/
    │   │   │   ├── model.config
    │   │   │   └── model.sdf   
    │   │   └── right_wheel/
    │   │       ├── model.config
    │   │       └── model.sdf  
    │   └── sensors/        
    │       └── imu/
    │           ├── model.config
    │           └── model.sdf   
    └── ground_plane/
        ├── model.config
        └── model.sdf
```

## 3. Joint Controller 플러그인 이해하기

Gazebo는 `JointController` 플러그인을 제공하여 각 조인트를 독립적으로 제어할 수 있게 합니다. 이 플러그인은 다음과 같은 기능을 제공합니다:

1. **PID 제어**: 비례(P), 적분(I), 미분(D) 제어 파라미터를 설정할 수 있습니다.
2. **다양한 제어 모드**: 위치, 속도, 힘/토크 등 다양한 제어 모드를 지원합니다.
3. **토픽 기반 제어**: ROS/Ignition 토픽을 통해 명령을 받습니다.

## 4. RPM 컨트롤러 구현

### 4.1 Joint Controller 플러그인 설정

메인 로봇 모델(`model.sdf`)에 Joint Controller 플러그인을 추가합니다:

```xml
<!-- RPM 기반 커스텀 컨트롤러 플러그인 -->
<plugin
  filename="libignition-gazebo-joint-controller-system.so"
  name="ignition::gazebo::systems::JointController">
  <!-- 좌측 바퀴 제어 -->
  <joint_name>left_wheel_joint</joint_name>
  <topic>/left_wheel/rpm</topic>
  <joint_type>velocity</joint_type>
  <p_gain>0.1</p_gain>
  <i_gain>0.01</i_gain>
  <d_gain>0.001</d_gain>
  <i_max>1</i_max>
  <i_min>-1</i_min>
  <cmd_max>5</cmd_max>
  <cmd_min>-5</cmd_min>
  <transform_velocity_to_rpm>true</transform_velocity_to_rpm>
</plugin>

<plugin
  filename="libignition-gazebo-joint-controller-system.so"
  name="ignition::gazebo::systems::JointController">
  <!-- 우측 바퀴 제어 -->
  <joint_name>right_wheel_joint</joint_name>
  <topic>/right_wheel/rpm</topic>
  <joint_type>velocity</joint_type>
  <p_gain>0.1</p_gain>
  <i_gain>0.01</i_gain>
  <d_gain>0.001</d_gain>
  <i_max>1</i_max>
  <i_min>-1</i_min>
  <cmd_max>5</cmd_max>
  <cmd_min>-5</cmd_min>
  <transform_velocity_to_rpm>true</transform_velocity_to_rpm>
</plugin>
```

주요 설정 파라미터 설명:
- `joint_name`: 제어할 조인트 이름
- `topic`: 명령을 받을 토픽 이름
- `joint_type`: 제어 모드(위치, 속도 등)
- `p_gain`, `i_gain`, `d_gain`: PID 제어기 게인
- `cmd_max`, `cmd_min`: 명령 값의 최대/최소 제한
- `transform_velocity_to_rpm`: 속도를 RPM 단위로 변환할지 여부

### 4.2 GUI에 RPM 제어 인터페이스 추가

Ignition Gazebo는 GUI를 통해 토픽에 메시지를 발행할 수 있는 Publisher 플러그인을 제공합니다. 이를 활용하여 슬라이더로 바퀴 RPM을 제어할 수 있는 인터페이스를 추가합니다:

```xml
<!-- RPM 제어용 Publisher 플러그인 -->
<plugin filename="Publisher" name="Left Wheel RPM">
  <gz-gui>
    <property key="showTitleBar" type="bool">true</property>
    <property key="resizable" type="bool">true</property>
    <property key="width" type="double">400</property>
    <property key="height" type="double">300</property>
    <property key="state" type="string">docked</property>
    <property key="title" type="string">Left Wheel RPM</property>
  </gz-gui>
  <topic>/left_wheel/rpm</topic>
  <message_type>ignition.msgs.Double</message_type>
  <default_value>0.0</default_value>
  <min>-60.0</min>
  <max>60.0</max>
</plugin>

<plugin filename="Publisher" name="Right Wheel RPM">
  <gz-gui>
    <property key="showTitleBar" type="bool">true</property>
    <property key="resizable" type="bool">true</property>
    <property key="width" type="double">400</property>
    <property key="height" type="double">300</property>
    <property key="state" type="string">docked</property>
    <property key="title" type="string">Right Wheel RPM</property>
  </gz-gui>
  <topic>/right_wheel/rpm</topic>
  <message_type>ignition.msgs.Double</message_type>
  <default_value>0.0</default_value>
  <min>-60.0</min>
  <max>60.0</max>
</plugin>
```

## 5. 시뮬레이션 실행 및 로봇 제어

다음 명령어로 시뮬레이션을 실행합니다:

```bash
cd ~/git/ros2_gazebo_ignition_tutorial
export IGN_GAZEBO_RESOURCE_PATH=$PWD
ign gazebo -r chapter8/diff_drive_world.sdf
```

### 5.1 GUI를 통한 RPM 제어

1. **좌/우 바퀴 RPM 설정**:
   - GUI에서 "Left Wheel RPM"과 "Right Wheel RPM" 슬라이더를 사용하여 각 바퀴의 RPM을 개별적으로 설정할 수 있습니다.
   - 양수 값: 전진 방향 회전
   - 음수 값: 후진 방향 회전
   - 값이 클수록 회전 속도가 빨라집니다.

2. **기본 이동 예시**:
   - **전진**: 두 바퀴 모두 동일한 양수 값 설정 (예: 좌측 30 RPM, 우측 30 RPM)
   - **후진**: 두 바퀴 모두 동일한 음수 값 설정 (예: 좌측 -30 RPM, 우측 -30 RPM)
   - **좌회전**: 우측 바퀴는 양수, 좌측 바퀴는 더 작은 양수 또는 0 설정 (예: 좌측 0 RPM, 우측 30 RPM)
   - **우회전**: 좌측 바퀴는 양수, 우측 바퀴는 더 작은 양수 또는 0 설정 (예: 좌측 30 RPM, 우측 0 RPM)
   - **제자리 회전**: 좌측 바퀴와 우측 바퀴를 반대 방향으로 설정 (예: 좌측 -30 RPM, 우측 30 RPM)

### 5.2 터미널에서 RPM 명령 발행

GUI 외에도 터미널에서 명령을 발행할 수 있습니다:

```bash
# 좌측 바퀴 30 RPM으로 설정
ign topic -t /left_wheel/rpm -m ignition.msgs.Double -p "data: 30.0"

# 우측 바퀴 30 RPM으로 설정
ign topic -t /right_wheel/rpm -m ignition.msgs.Double -p "data: 30.0"

# 좌측 바퀴 정지
ign topic -t /left_wheel/rpm -m ignition.msgs.Double -p "data: 0.0"

# 우측 바퀴 정지
ign topic -t /right_wheel/rpm -m ignition.msgs.Double -p "data: 0.0"
```

## 6. PID 제어기 조정

PID 제어기 파라미터는 로봇 성능에 큰 영향을 미칩니다. 필요에 따라 다음 값들을 조정할 수 있습니다:

- **P 게인 (p_gain)**: 현재 오차에 비례하는 제어 입력. 큰 값은 빠른 응답을 제공하지만 오버슈트가 발생할 수 있습니다.
- **I 게인 (i_gain)**: 오차의 누적에 비례하는 제어 입력. 정상 상태 오차를 줄여줍니다.
- **D 게인 (d_gain)**: 오차의 변화율에 비례하는 제어 입력. 오버슈트를 줄이고 안정성을 높여줍니다.

파라미터 조정 예시:
- 응답이 너무 느리면: P 게인을 증가
- 정상 상태 오차가 있으면: I 게인을 증가
- 오버슈트나 진동이 발생하면: D 게인을 증가하거나 P 게인을 감소

## 7. 응용: 커스텀 제어 알고리즘 구현

RPM 기반 제어를 활용하여 다양한 커스텀 제어 알고리즘을 구현할 수 있습니다:

### 7.1 외부 제어 스크립트 예시 (Python)

```python
#!/usr/bin/env python3
import time
import math
from ignition.transport import Node

# Node 초기화
node = Node()

# 발행자 생성
left_pub = node.advertise("/left_wheel/rpm", "ignition.msgs.Double")
right_pub = node.advertise("/right_wheel/rpm", "ignition.msgs.Double")

# Double 메시지 생성 함수
def create_double_msg(value):
    msg = node.create_message("ignition.msgs.Double")
    msg.data = value
    return msg

# 원형 경로 주행
def drive_circle(radius, speed, duration):
    start_time = time.time()
    
    while time.time() - start_time < duration:
        # 원형 경로를 위한 좌우 바퀴 RPM 계산
        # 바퀴 간격이 0.35m일 경우
        left_rpm = speed * (1 - 0.35 / (2 * radius))
        right_rpm = speed * (1 + 0.35 / (2 * radius))
        
        # RPM 명령 발행
        left_pub.publish(create_double_msg(left_rpm))
        right_pub.publish(create_double_msg(right_rpm))
        
        time.sleep(0.01)
    
    # 정지
    left_pub.publish(create_double_msg(0.0))
    right_pub.publish(create_double_msg(0.0))

# 1m 반경, 30 RPM 속도로 10초간 원형 경로 주행
drive_circle(1.0, 30.0, 10.0)
```

### 7.2 IMU 센서 데이터를 활용한 방향 제어

```python
#!/usr/bin/env python3
import time
import math
from ignition.transport import Node

# Node 초기화
node = Node()

# 발행자 생성
left_pub = node.advertise("/left_wheel/rpm", "ignition.msgs.Double")
right_pub = node.advertise("/right_wheel/rpm", "ignition.msgs.Double")

# IMU 데이터 콜백
current_yaw = 0.0

def imu_callback(msg):
    global current_yaw
    # IMU 쿼터니언에서 yaw 각도 추출
    qw = msg.orientation.w
    qx = msg.orientation.x
    qy = msg.orientation.y
    qz = msg.orientation.z
    current_yaw = math.atan2(2.0 * (qw * qz + qx * qy), 
                             1.0 - 2.0 * (qy * qy + qz * qz))

# IMU 토픽 구독
node.subscribe("/imu", "ignition.msgs.IMU", imu_callback)

# 특정 방향으로 로봇 회전
def rotate_to_target(target_yaw, speed=20.0):
    global current_yaw
    
    while abs(current_yaw - target_yaw) > 0.05:
        # 방향에 따라 회전 방향 결정
        error = target_yaw - current_yaw
        if error > math.pi:
            error -= 2 * math.pi
        elif error < -math.pi:
            error += 2 * math.pi
            
        # PID 제어 (여기서는 단순한 P 제어만 사용)
        p_gain = 10.0
        rot_speed = p_gain * error
        
        # 속도 제한
        if rot_speed > speed:
            rot_speed = speed
        elif rot_speed < -speed:
            rot_speed = -speed
            
        # RPM 명령 발행
        left_pub.publish(create_double_msg(-rot_speed))
        right_pub.publish(create_double_msg(rot_speed))
        
        time.sleep(0.01)
    
    # 정지
    left_pub.publish(create_double_msg(0.0))
    right_pub.publish(create_double_msg(0.0))

# 90도(π/2) 방향으로 회전
rotate_to_target(math.pi/2)
```

## 8. 문제 해결

- **바퀴가 회전하지 않음**: 
  - Joint Controller 플러그인 설정의 `joint_name`이 실제 로봇 모델의 조인트 이름과 일치하는지 확인하세요.
  - 토픽 이름이 올바른지 확인하세요.
  - PID 게인 값이 너무 작지 않은지 확인하세요.

- **바퀴 회전이 불안정함**: 
  - PID 게인을 조정하세요. 특히 D 게인을 증가시키면 안정성이 향상될 수 있습니다.
  - `cmd_max`와 `cmd_min` 값을 확인하세요. 너무 큰 값은 불안정한 동작을 유발할 수 있습니다.

- **RPM 명령이 반영되지 않음**: 
  - 터미널에서 `ign topic -l | grep rpm`을 실행하여 RPM 토픽이 등록되어 있는지 확인하세요.
  - `ign topic -e -t /left_wheel/rpm`을 실행하여 실제 명령이 발행되고 있는지 확인하세요.

## 다음 단계

이제 RPM 기반 커스텀 컨트롤러를 구현하고 사용하는 방법을 배웠습니다. 다음 장에서는 이러한 제어 시스템을 ROS2와 통합하여 더 복잡한 로봇 애플리케이션을 개발하는 방법을 알아보겠습니다. 