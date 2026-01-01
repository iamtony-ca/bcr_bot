완벽하게 정리된 Xacro 파일들을 바탕으로, **ROS 2 Jazzy(Gazebo Harmonic)**와 **Isaac Sim**에서 실제로 로봇을 띄우고 활용하는 방법을 단계별로 안내해 드리겠습니다.

---

### 📍 Step 0: 패키지 환경 구축 (파일 배치)

먼저 ROS 2 워크스페이스에 패키지를 만들고 파일을 제 위치에 두어야 합니다.

1. **패키지 생성:**
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake bcr_bot
cd bcr_bot
mkdir -p urdf meshes launch

```


2. **파일 저장:**
아래 파일들을 `~/ros2_ws/src/bcr_bot/urdf/` 경로에 저장하세요.
* `bcr_bot.xacro` (Main)
* `macros.xacro`
* `materials.xacro`
* `gz.xacro`


3. **메쉬 파일 확인:**
코드에 `meshes/bcr_bot_mesh.dae` 등이 참조되어 있습니다. 해당 `.dae` 또는 `.stl` 파일들을 `~/ros2_ws/src/bcr_bot/meshes/` 폴더에 넣어야 합니다.
*(만약 파일이 없다면 Xacro에서 `<mesh ...>` 부분을 `<box size="..."/>` 등으로 임시 변경해야 에러가 안 납니다.)*
4. **빌드 및 소싱:**
```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select bcr_bot
source install/setup.bash

```



---

### 📍 Step 1: 시뮬레이터별 URDF 생성 (핵심)

하나의 Xacro에서 두 가지 버전의 URDF를 뽑아냅니다.

```bash
# 패키지 경로로 이동
cd ~/ros2_ws/src/bcr_bot/urdf

# 1. Gazebo Harmonic용 URDF 생성 (플러그인 포함)
xacro bcr_bot.xacro target_sim:=gazebo > bcr_bot_gazebo.urdf

# 2. Isaac Sim용 URDF 생성 (순수 형상 정보)
xacro bcr_bot.xacro target_sim:=isaac > bcr_bot_isaac.urdf

```

---

### 📍 Step 2: Gazebo Harmonic (Jazzy)에서 실행

Gazebo는 `ros_gz_sim`을 통해 환경을 열고 로봇을 스폰(Spawn)합니다.

1. **Gazebo 실행 (빈 월드):**
```bash
# 터미널 1
ros2 launch ros_gz_sim gz_sim.launch.py gz_args:="-r empty.sdf"

```


2. **로봇 스폰 (Spawn):**
방금 만든 `bcr_bot_gazebo.urdf` 내용을 읽어서 Gazebo 안에 집어넣습니다.
```bash
# 터미널 2
cd ~/ros2_ws/src/bcr_bot/urdf
ros2 run ros_gz_sim create -file bcr_bot_gazebo.urdf -name bcr_bot -z 0.5

```


3. **동작 확인:**
* 로봇이 화면에 나타나면 성공입니다.
* **제어 테스트:** 다른 터미널에서 `/cmd_vel` 토픽을 발행해 로봇이 움직이는지 확인합니다.


```bash
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.5}, angular: {z: 0.5}}"

```



---

### 📍 Step 3: NVIDIA Isaac Sim에서 실행

Isaac Sim은 URDF를 "Import" 하여 USD로 변환한 뒤 사용합니다.

1. **Isaac Sim 실행:**
Isaac Sim을 실행합니다.
2. **URDF Importer 실행:**
* 상단 메뉴바: **Isaac Utils** > **Workflows** > **URDF Importer** 선택.


3. **Import 설정:**
* **Input File:** `~/ros2_ws/src/bcr_bot/urdf/bcr_bot_isaac.urdf` 선택.
* **Import Settings:**
* `Clean Mesh`: 체크 권장.
* `Fix Base Link`: 체크 해제 (모바일 로봇이므로 고정되면 안 됨).
* `Joint Drive Strength`: 기본값 유지 (나중에 튜닝).
* `Self Collision`: 필요 시 체크 (Xacro에 설정했으므로 URDF 정보 따름).


* **Import 버튼 클릭.**


4. **USD 저장 및 활용:**
* 로봇이 로드되면 `Stage` 패널에서 로봇(Prim)을 우클릭하고 **"Save As..."**를 눌러 `bcr_bot.usd`로 저장합니다.
* 이제 이 `.usd` 파일을 드래그 앤 드롭하여 어떤 환경(Warehouse, Office 등)에서도 로봇을 불러올 수 있습니다.



---

### 💡 (심화) Isaac Sim에서 ROS 2 제어 연결하기

Isaac Sim에 로봇을 불러왔다고 바로 `/cmd_vel`로 움직이지 않습니다. **Action Graph**를 구성해야 합니다.

1. **Omnigraph 열기:** Window > Visual Scripting > Action Graph.
2. **노드 추가:**
* `ROS2 Context`: ROS 2 통신 시작.
* `ROS2 Subscribe Twist`: `/cmd_vel` 토픽 수신.
* `Differential Controller`: Twist 메시지를 휠 속도로 변환.
* `Articulation Controller`: 휠 조인트에 힘(Velocity) 전달.


3. **연결:** `Subscribe Twist` -> `Differential Controller` -> `Articulation Controller` 순으로 연결하고, 대상 로봇(Target Prim)을 지정해 줍니다.

---

### 🚀 요약 및 Next Step

이제 **하나의 소스(`bcr_bot.xacro`)**로:

1. **Gazebo:** `make target_sim:=gazebo` -> `ros2 run create`
2. **Isaac Sim:** `make target_sim:=isaac` -> `Import & Save USD`

이렇게 완벽하게 이원화된 워크플로우를 갖추셨습니다.

**다음 단계로 무엇을 도와드릴까요?**

1. **Isaac Sim의 Action Graph 구성 방법**을 자세히 알고 싶으신가요? (로봇을 실제로 움직이게 하기 위해)
2. **Gazebo Launch 파일(.py)**을 만들어서 `robot_state_publisher`와 RViz까지 한 번에 켜는 스크립트가 필요하신가요?