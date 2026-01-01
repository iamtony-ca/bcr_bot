### 🔥 생성 및 활용 방법

이제 터미널에서 아래 명령어로 목적에 맞는 URDF를 각각 생성할 수 있습니다.

#### 1. Gazebo Harmonic (Jazzy)용 생성

이 파일은 `ros_gz` 스포너에 넣으시면 됩니다.

```bash
xacro urdf/bcr_bot.xacro target_sim:=gazebo > bcr_bot_gazebo.urdf

```

#### 2. Isaac Sim Import용 생성

이 파일은 Isaac Sim의 URDF Importer에서 불러오시면 됩니다. 불필요한 플러그인 태그가 없어서 Import 오류가 발생하지 않고, 바로 USD로 변환하기 최적화된 상태가 됩니다.

```bash
xacro urdf/bcr_bot.xacro target_sim:=isaac > bcr_bot_isaac.urdf

```

이렇게 하면 하나의 Xacro 파일 세트로 두 시뮬레이터를 완벽하게 지원할 수 있습니다!