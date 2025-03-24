# 1장: 환경 설정 및 Gazebo GUI 실행하기

이 챕터에서는 ROS2 Humble과 Gazebo Ignition을 설치하고 기본적인 Gazebo GUI를 실행하는 방법을 알아봅니다.

## ROS2 Humble 설치

1. ROS2 저장소 설정
```bash
sudo apt update && sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

2. ROS2 Humble 설치
```bash
sudo apt update
sudo apt install ros-humble-desktop
```

3. 환경 설정
```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## ROS2와 Gazebo Ignition 연동 설치

ROS2 Humble에서는 `ros-humble-ros-gz` 메타패키지를 통해 Gazebo Ignition과의 모든 연동 패키지를 설치할 수 있습니다.

1. ros-gz 패키지 설치
```bash
sudo apt install ros-humble-ros-gz
```

이 메타패키지를 설치하면 다음 패키지들이 모두 함께 설치됩니다:
- ros-humble-ros-gz-bridge: ROS2와 Gazebo 간의 메시지 변환 브릿지
- ros-humble-ros-gz-interfaces: 인터페이스 정의
- ros-humble-ros-gz-sim: Gazebo Simulation과 ROS2 연동
- ros-humble-ros-gz-image: 이미지 관련 연동
- ros-humble-ros-gz-point-cloud: 포인트 클라우드 연동

2. 환경 설정
```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## SDF 파일 생성 및 Gazebo GUI 실행하기

SDF(Simulation Description Format)는 시뮬레이션 환경을 정의하는 XML 형식의 파일입니다. 이 챕터에서는 간단한 빈 월드를 정의하는 SDF 파일을 생성하고 실행해보겠습니다.

1. SDF 파일 구조 이해하기

`empty_world.sdf` 파일은 다음과 같은 구성 요소를 포함합니다:

- `<world>`: 시뮬레이션 월드 정의
- `<physics>`: 물리 시뮬레이션 특성 정의
- `<plugin>`: 기능 확장을 위한 플러그인
- `<light>`: 조명 정의
- `<model>`: 바닥 평면 모델 정의

2. SDF 파일 살펴보기

`empty_world.sdf` 파일을 텍스트 에디터로 열어봅시다:

```bash
nano empty_world.sdf
# 또는
gedit empty_world.sdf
```

3. Gazebo로 SDF 파일 실행하기

SDF 파일을 작성한 후 Gazebo에서 실행할 수 있습니다:

```bash
# 현재 디렉토리의 SDF 파일 실행
ign gazebo -r empty_world.sdf

# 또는 절대 경로 사용
ign gazebo -r /full/path/to/empty_world.sdf
```

`-r` 옵션은 시뮬레이션을 바로 실행(run)하라는 의미입니다.

4. 주요 옵션 설명
- `-r` 또는 `--run`: 시뮬레이션을 바로 실행 (자동 재생)
- `-g` 또는 `--gui-config`: GUI 설정 파일 지정
- `-v` 또는 `--verbose`: 자세한 로그 출력
- `-s` 또는 `--server`: 서버 모드로 실행 (GUI 없음)

## GUI 인터페이스 탐색하기

Gazebo GUI가 실행되면 다음과 같은 주요 인터페이스 요소를 볼 수 있습니다:

1. 3D 뷰: 시뮬레이션 환경을 보여주는 메인 창
2. 왼쪽 패널: 엔티티 트리, 모델, 라이트 등을 표시
3. 상단 툴바: 이동, 회전, 스케일 등의 도구
4. 하단 타임라인: 시뮬레이션 제어 (재생, 일시정지, 단계 실행)

마우스를 사용하여 다음과 같은 조작이 가능합니다:
- 왼쪽 버튼 드래그: 화면 회전
- 휠 버튼 드래그: 화면 이동
- 휠 스크롤: 화면 확대/축소
- 오른쪽 버튼 클릭: 컨텍스트 메뉴 열기

## 문제 해결

만약 Gazebo GUI가 실행되지 않는다면 다음을 확인해주세요:

1. SDF 파일 문법 확인
```bash
xmllint --noout empty_world.sdf
```

2. Gazebo 설치 확인
```bash
ign --version
```

3. 환경 변수 설정 확인
```bash
printenv | grep ROS
printenv | grep IGN
```

4. 그래픽 관련 문제 해결
```bash
# NVIDIA GPU 사용 시 드라이버 확인
nvidia-smi
# DISPLAY 환경 변수 확인
echo $DISPLAY
```

## 다음 단계

이제 기본적인 환경 설정이 완료되었고 직접 만든 SDF 파일로 Gazebo 시뮬레이션을 실행해보았습니다. 다음 챕터에서는 ROS2와 Gazebo Ignition을 연동하여 간단한 로봇을 시뮬레이션하는 방법을 알아보겠습니다. 