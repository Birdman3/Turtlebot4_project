# 환경과 명령 실행 위치

## 확인된 환경

| 항목 | 확인 내용 |
|---|---|
| 로봇 | TurtleBot 4, 세부 모델은 확인하지 않음 |
| 로봇 ROS 2 배포판 | Humble |
| 로봇 DDS 구현 | `rmw_fastrtps_cpp` |
| 로봇 Domain ID | `0` |
| 로봇 Discovery Server | ID `0`, UDP 포트 `11811` |
| 로봇 로컬 연결 주소 | `127.0.0.1:11811;` |
| 사용자 PC | Windows + Ubuntu/WSL |
| WSL 버전 / PC ROS 설치 상태 | 별도 확인 필요 |

## 반드시 구분할 세 터미널

| 위치 | 실행할 명령 예 | 설명 |
|---|---|---|
| 로봇에 SSH로 접속한 터미널 | `systemctl status turtlebot4.service` | 명령은 로봇에서 실행됨 |
| 사용자 PC의 Ubuntu/WSL 로컬 터미널 | `wget ...`, PC의 `ros2 topic list` | 명령은 PC의 Linux 환경에서 실행됨 |
| Windows PowerShell | `Resolve-DnsName ...`, `wsl --shutdown` | 명령은 Windows에서 실행됨 |

VS Code 터미널처럼 생김새가 같아도 실행 위치는 다를 수 있습니다. SSH 접속 여부를 먼저 확인합니다.

## 로봇 터미널에서 관찰한 환경변수

```text
ROS_VERSION=2
ROS_PYTHON_VERSION=3
ROS_DISTRO=humble
ROS_DOMAIN_ID=0
ROS_LOCALHOST_ONLY=0
RMW_IMPLEMENTATION=rmw_fastrtps_cpp
ROS_DISCOVERY_SERVER=127.0.0.1:11811;
ROS_SUPER_CLIENT=True
FASTRTPS_DEFAULT_PROFILES_FILE=/etc/turtlebot4/fastdds_rpi.xml
CYCLONEDDS_URI=/etc/turtlebot4/cyclonedds_rpi.xml
```

`CYCLONEDDS_URI`가 존재한다는 사실만으로 CycloneDDS가 사용 중이라고 판단하면 안 됩니다. 해당 화면의 선택된 구현은 `RMW_IMPLEMENTATION=rmw_fastrtps_cpp`입니다.

`ROS_LOCALHOST_ONLY=0`은 localhost로만 제한하는 설정이 꺼졌다는 뜻이지, 네트워크 전체가 정상이라는 보장은 아닙니다.

## 주소와 설정 파일

| 대상 | 설정 |
|---|---|
| 로봇 자신이 서버에 연결 | `127.0.0.1:11811;` |
| PC가 로봇 서버에 연결 | `<ROBOT_WIFI_IP>:11811;` |
| 로봇 환경 파일 | `/etc/turtlebot4/setup.bash` |
| 로봇 서버 실행 스크립트 | `/etc/turtlebot4/discovery.sh` |
| PC 설정 스크립트가 만드는 환경 파일 | `/etc/turtlebot4_discovery/setup.bash` |

PC에서 `127.0.0.1`은 PC 자신을 가리킵니다. PC 설정에는 현재 로봇의 Wi-Fi IP를 사용합니다. 로봇에 SSH로 접속한 상태에서 `ip -4 addr show wlan0`로 주소를 확인할 수 있습니다.

[공식 Discovery Server 문서](https://turtlebot.github.io/turtlebot4-user-manual/setup/discovery_server.html)
