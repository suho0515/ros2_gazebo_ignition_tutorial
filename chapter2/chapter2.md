# 2장: 객체 모델링 및 월드에 추가하기

이 챕터에서는 객체를 모델링하고 시뮬레이션 월드에 추가하는 방법을 알아봅니다.

## SDF로 3D 객체 모델링

SDF(Simulation Description Format)는 로봇, 정적 객체, 동적 객체 등의 모델을 정의하는 XML 기반 형식입니다. 이 챕터에서는 다음 세 가지 기본 객체를 모델링하겠습니다:

1. 빨간색 박스 (box.sdf)
2. 파란색 구체 (sphere.sdf)
3. 녹색 실린더 (world_with_objects.sdf 내부에 직접 정의)

### 박스 모델 생성하기

`box.sdf` 파일은 1x1x1 크기의 빨간색 박스를 정의합니다:

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <model name="box">
    <pose>0 0 0.5 0 0 0</pose>
    <link name="box_link">
      <inertial>
        <mass>1.0</mass>
        <inertia>
          <ixx>0.166667</ixx>
          <ixy>0</ixy>
          <ixz>0</ixz>
          <iyy>0.166667</iyy>
          <iyz>0</iyz>
          <izz>0.166667</izz>
        </inertia>
      </inertial>
      <collision name="box_collision">
        <geometry>
          <box>
            <size>1 1 1</size>
          </box>
        </geometry>
      </collision>
      <visual name="box_visual">
        <geometry>
          <box>
            <size>1 1 1</size>
          </box>
        </geometry>
        <material>
          <ambient>1 0 0 1</ambient>
          <diffuse>1 0 0 1</diffuse>
          <specular>1 0 0 1</specular>
        </material>
      </visual>
    </link>
  </model>
</sdf>
```

주요 구성 요소:
- `<inertial>`: 질량과 관성 모멘트 정의
- `<collision>`: 물리 충돌 처리를 위한 형상 정의
- `<visual>`: 시각적 표현을 위한 형상과 재질 정의

### 구체 모델 생성하기

`sphere.sdf` 파일은 반지름 0.5의 파란색 구체를 정의합니다:

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <model name="sphere">
    <pose>2 0 0.5 0 0 0</pose>
    <link name="sphere_link">
      <inertial>
        <mass>1.0</mass>
        <inertia>
          <ixx>0.1</ixx>
          <ixy>0</ixy>
          <ixz>0</ixz>
          <iyy>0.1</iyy>
          <iyz>0</iyz>
          <izz>0.1</izz>
        </inertia>
      </inertial>
      <collision name="sphere_collision">
        <geometry>
          <sphere>
            <radius>0.5</radius>
          </sphere>
        </geometry>
      </collision>
      <visual name="sphere_visual">
        <geometry>
          <sphere>
            <radius>0.5</radius>
          </sphere>
        </geometry>
        <material>
          <ambient>0 0 1 1</ambient>
          <diffuse>0 0 1 1</diffuse>
          <specular>0 0 1 1</specular>
        </material>
      </visual>
    </link>
  </model>
</sdf>
```

## 객체를 월드에 포함하기

이제 개별 객체 모델을 월드에 통합해 보겠습니다. 새로운 월드 파일(`world_with_objects.sdf`)을 만들고 세 가지 객체를 포함시킬 것입니다.

### 월드 파일 생성하기

`world_with_objects.sdf` 파일을 생성하고 다음 내용을 추가합니다:

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="world_with_objects">
    <physics name="1ms" type="ignored">
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1.0</real_time_factor>
    </physics>
    <plugin
      filename="ignition-gazebo-physics-system"
      name="ignition::gazebo::systems::Physics">
    </plugin>
    <plugin
      filename="ignition-gazebo-user-commands-system"
      name="ignition::gazebo::systems::UserCommands">
    </plugin>
    <plugin
      filename="ignition-gazebo-scene-broadcaster-system"
      name="ignition::gazebo::systems::SceneBroadcaster">
    </plugin>

    <light type="directional" name="sun">
      <cast_shadows>true</cast_shadows>
      <pose>0 0 10 0 0 0</pose>
      <diffuse>0.8 0.8 0.8 1</diffuse>
      <specular>0.2 0.2 0.2 1</specular>
      <attenuation>
        <range>1000</range>
        <constant>0.9</constant>
        <linear>0.01</linear>
        <quadratic>0.001</quadratic>
      </attenuation>
      <direction>-0.5 0.1 -0.9</direction>
    </light>

    <model name="ground_plane">
      <static>true</static>
      <link name="link">
        <collision name="collision">
          <geometry>
            <plane>
              <normal>0 0 1</normal>
              <size>100 100</size>
            </plane>
          </geometry>
        </collision>
        <visual name="visual">
          <geometry>
            <plane>
              <normal>0 0 1</normal>
              <size>100 100</size>
            </plane>
          </geometry>
          <material>
            <ambient>0.8 0.8 0.8 1</ambient>
            <diffuse>0.8 0.8 0.8 1</diffuse>
            <specular>0.8 0.8 0.8 1</specular>
          </material>
        </visual>
      </link>
    </model>

    <!-- 빨간색 박스 추가 -->
    <include>
      <uri>box.sdf</uri>
      <pose>0 0 0.5 0 0 0</pose>
      <name>red_box</name>
    </include>

    <!-- 파란색 구체 추가 -->
    <include>
      <uri>sphere.sdf</uri>
      <pose>2 0 0.5 0 0 0</pose>
      <name>blue_sphere</name>
    </include>

    <!-- 녹색 실린더 추가 -->
    <model name="green_cylinder">
      <pose>-2 0 0.5 0 0 0</pose>
      <link name="cylinder_link">
        <inertial>
          <mass>1.0</mass>
          <inertia>
            <ixx>0.1458</ixx>
            <ixy>0</ixy>
            <ixz>0</ixz>
            <iyy>0.1458</iyy>
            <iyz>0</iyz>
            <izz>0.125</izz>
          </inertia>
        </inertial>
        <collision name="cylinder_collision">
          <geometry>
            <cylinder>
              <radius>0.5</radius>
              <length>1.0</length>
            </cylinder>
          </geometry>
        </collision>
        <visual name="cylinder_visual">
          <geometry>
            <cylinder>
              <radius>0.5</radius>
              <length>1.0</length>
            </cylinder>
          </geometry>
          <material>
            <ambient>0 1 0 1</ambient>
            <diffuse>0 1 0 1</diffuse>
            <specular>0 1 0 1</specular>
          </material>
        </visual>
      </link>
    </model>
  </world>
</sdf>
```

이 파일에서는:
1. `<include>` 태그를 사용하여 외부 SDF 파일(box.sdf, sphere.sdf)을 참조합니다.
2. `<pose>` 태그로 각 객체의 위치를 지정합니다.
3. `<name>` 태그로 포함된 모델의 이름을 지정합니다.
4. 녹색 실린더는 월드 파일 내에 직접 정의되었습니다.

## 시뮬레이션 실행하기

세 가지 객체가 포함된 월드를 시뮬레이션하려면 다음 명령어를 사용합니다:

```bash
cd chapter2
ign gazebo -r world_with_objects.sdf
```

> **주의사항**: SDF 파일의 참조 경로 문제가 발생할 수 있습니다. `<include>` 태그의 `<uri>` 요소는 상대 경로나 모델 데이터베이스 경로일 수 있습니다. 경로 문제가 있는 경우 절대 경로를 사용해볼 수 있습니다.

### 객체 조작 방법

1. 객체 이동
   - 마우스로 객체를 선택하고 드래그합니다.
   - 키보드 화살표 키를 사용하여 선택한 객체를 이동합니다.

2. 물리 속성 관찰하기
   - 객체가 떨어지는 것을 관찰합니다.
   - 객체 간 충돌을 관찰합니다.

## SDF 파일 모델 디렉토리에 저장하기

객체 모델을 모델 데이터베이스에 추가하여 다른 SDF 파일에서 쉽게 참조할 수 있습니다:

```bash
mkdir -p ~/.gazebo/models/box
mkdir -p ~/.gazebo/models/sphere
cp box.sdf ~/.gazebo/models/box/model.sdf
cp sphere.sdf ~/.gazebo/models/sphere/model.sdf
```

또한 각 모델 디렉토리에 `model.config` 파일을 생성하여 메타데이터를 추가할 수 있습니다:

```xml
<?xml version="1.0"?>
<model>
  <name>Box</name>
  <version>1.0</version>
  <sdf version="1.6">model.sdf</sdf>
  <author>
    <name>Your Name</name>
    <email>your.email@example.com</email>
  </author>
  <description>A simple box model</description>
</model>
```

## 문제 해결

**문제 1: <include> 태그의 이름 태그 문제**

SDF 파일 생성 시 `<name>` 태그 대신 `<n>` 태그가 사용될 수 있습니다. 수동으로 파일을 편집하여 수정하세요:

```bash
sed -i 's/<n>/<name>/g' world_with_objects.sdf
sed -i 's/<\/n>/<\/name>/g' world_with_objects.sdf
```

**문제 2: 참조된 모델을 찾을 수 없음**

상대 경로 문제가 발생하면 다음과 같이 절대 경로를 사용해보세요:

```xml
<include>
  <uri>file:///full/path/to/box.sdf</uri>
  <pose>0 0 0.5 0 0 0</pose>
  <name>red_box</name>
</include>
```

## 다음 단계

이제 객체를 모델링하고 시뮬레이션 월드에 추가하는 방법을 배웠습니다. 다음 챕터에서는 간단한 로봇을 모델링하고 ROS2를 통해 제어하는 방법을 알아보겠습니다. 