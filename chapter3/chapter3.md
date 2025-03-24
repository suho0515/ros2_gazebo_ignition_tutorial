# 3장: 모듈식 SDF 파일 구조화

이 챕터에서는 SDF 파일을 모듈화하여 구조적으로 관리하는 방법을 알아봅니다. 대규모 시뮬레이션 환경을 구성할 때 이러한 모듈화 접근 방식은 유지 관리와 재사용성을 크게 향상시킵니다.

## 모듈식 디렉토리 구조

Gazebo Ignition의 SDF 파일 구조를 효율적으로 관리하기 위해 다음과 같은 디렉토리 구조를 사용합니다:

```
chapter3/
├── model/          # 재사용 가능한 모델 파일
│   ├── ground_plane.sdf   # 지면 모델
│   ├── box.sdf            # 박스 모델
│   ├── sphere.sdf         # 구체 모델
│   └── ...                # 추가 모델
└── world.sdf       # 모든 요소를 통합하는 메인 월드 파일
```

이 구조의 특징:
- 모델 파일만 별도로 분리하여 재사용성을 높입니다.
- 물리, 플러그인, 조명 설정은 world.sdf 파일에 직접 포함합니다.
- 모델 파일은 필요에 따라 다른 프로젝트에서도 재사용할 수 있습니다.

> **중요**: Gazebo Ignition에서는 물리 설정, 플러그인, 조명 등의 설정을 별도 파일로 분리하여 `<include>` 태그로 포함하는 것이 올바르게 작동하지 않습니다. 이러한 설정은 world 파일에 직접 작성해야 합니다.

## 1. 월드 파일의 구성 요소

### 1.1 물리 시뮬레이션 설정 (Physics)

물리 시뮬레이션 설정은 `world.sdf` 파일 내에 직접 정의합니다:

```xml
<physics name="sim_physics" type="ode">
  <max_step_size>0.001</max_step_size>
  <real_time_factor>1.0</real_time_factor>
  <real_time_update_rate>1000</real_time_update_rate>
  <gravity>0 0 -9.8</gravity>
  <ode>
    <!-- 물리 엔진 세부 설정 -->
  </ode>
</physics>
```

주요 물리 설정 요소:
- `<max_step_size>`: 시뮬레이션 단계의 최대 시간 간격
- `<real_time_factor>`: 실시간 대비 시뮬레이션 시간 비율
- `<gravity>`: 중력 벡터
- `<ode>`: Open Dynamics Engine 설정
  - `<solver>`: 물리 충돌 해결기 설정
  - `<constraints>`: 물리 제약 조건 설정

### 1.2 플러그인 설정 (Plugin)

시뮬레이션에 필요한 플러그인도 `world.sdf` 파일 내에 직접 정의합니다:

```xml
<plugin
  filename="ignition-gazebo-physics-system"
  name="ignition::gazebo::systems::Physics">
</plugin>
<!-- 추가 플러그인 -->
```

필수 플러그인:
- **Physics**: 물리 시뮬레이션 기능 제공
- **UserCommands**: 사용자 명령 처리 기능
- **SceneBroadcaster**: 장면 정보 브로드캐스팅
- **Sensors**: 센서 시뮬레이션 기능 (렌더링 엔진 포함)

### 1.3 조명 설정 (Light)

조명 설정 역시 `world.sdf` 파일 내에 직접 정의합니다:

```xml
<light type="directional" name="sun">
  <cast_shadows>true</cast_shadows>
  <pose>0 0 10 0 0 0</pose>
  <!-- 추가 조명 설정 -->
</light>
```

조명 유형:
- **Directional Light (방향성 조명)**: 태양과 같은 광원
- **Spot Light (스포트 라이트)**: 원뿔 형태의 광원
- **Point Light (점 조명)**: 모든 방향으로 빛을 발산하는 광원

## 2. 모델 정의 (Model)

모델 파일은 모듈식으로 분리하여 관리할 수 있습니다. 이는 재사용성을 높이고 시뮬레이션 환경을 효율적으로 구성하는데 도움이 됩니다.

예를 들어, `model/box.sdf` 파일은 다음과 같이 구성됩니다:

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <model name="box">
    <!-- 모델 정의 -->
  </model>
</sdf>
```

각 모델 파일은 반드시 하나의 `<model>` 요소만 포함해야 합니다.

## 3. 올바른 월드 파일 구성 방법

올바른 월드 파일 구성 예시:

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="modular_world">
    <!-- 물리 설정 직접 정의 -->
    <physics name="sim_physics" type="ode">
      <!-- 물리 설정 내용 -->
    </physics>
    
    <!-- 플러그인 직접 정의 -->
    <plugin
      filename="ignition-gazebo-physics-system"
      name="ignition::gazebo::systems::Physics">
    </plugin>
    <!-- 추가 플러그인... -->
    
    <!-- 조명 직접 정의 -->
    <light type="directional" name="sun">
      <!-- 조명 설정... -->
    </light>
    
    <!-- 모델 파일만 include 사용 -->
    <include>
      <uri>file://model/ground_plane.sdf</uri>
    </include>
    
    <include>
      <uri>file://model/box.sdf</uri>
      <name>red_box</name>
      <pose>3 0 0.5 0 0 0</pose>
    </include>
    
    <!-- 추가 모델... -->
  </world>
</sdf>
```

## 4. 모듈식 SDF 사용 시 주의사항

Gazebo Ignition의 `<include>` 태그 사용 시 다음 사항에 유의해야 합니다:

1. **올바른 엔티티 포함**: `<include>` 태그는 오직 단일 최상위 엔티티(`<model>`, `<actor>`, `<light>`)를 포함하는 데만 사용해야 합니다.

2. **물리, 플러그인, 조명 설정 통합 방법**: 
   - 물리 설정, 플러그인, 조명은 반드시 `world.sdf` 파일에 직접 정의해야 합니다.
   - `<include>` 태그로 이러한 설정 파일을 포함하려고 하면 오류가 발생합니다.

3. **모델 식별자 지정**: 
   - 동일한 모델을 여러 번 포함할 때는 `<name>` 태그로 고유 식별자를 지정해야 합니다.
   - 위치는 `<pose>` 태그로 지정합니다.

## 5. 시뮬레이션 실행

모듈식 월드를 실행하려면 다음 명령어를 사용합니다:

```bash
cd chapter3
ign gazebo -r world.sdf
```

> **주의사항**: 모델 파일이 올바르게 참조되지 않는 경우, 절대 경로를 사용하거나 환경 변수를 설정하여 문제를 해결할 수 있습니다.

## 6. 경로 문제 해결

모듈식 구조에서 참조 경로 문제가 발생할 경우, 다음 두 가지 해결 방법이 있습니다:

### 6.1 절대 경로 사용

```xml
<include>
  <uri>file:///full/path/to/chapter3/model/box.sdf</uri>
</include>
```

### 6.2 IGN_GAZEBO_RESOURCE_PATH 환경 변수 설정

```bash
export IGN_GAZEBO_RESOURCE_PATH=$IGN_GAZEBO_RESOURCE_PATH:/path/to/your/project
```

## 7. 모듈식 구조의 장점

1. **재사용성**: 모델 파일을 여러 시뮬레이션에서 재사용할 수 있습니다.
2. **유지 관리성**: 각 모델을 독립적으로 업데이트할 수 있습니다.
3. **협업**: 팀원들이 서로 다른 모델을 병렬로 개발할 수 있습니다.
4. **가독성**: 파일이 작고 집중된 목적을 가지므로 이해하기 쉽습니다.
5. **확장성**: 새 모델을 쉽게 추가할 수 있습니다.

## 다음 단계

이제 모듈식 SDF 파일 구조를 생성하고 관리하는 방법을 배웠습니다. 다음 챕터에서는 모듈식 구조를 활용하여 더 복잡한 로봇 모델을 만들고 ROS2와 통합하는 방법을 알아보겠습니다. 