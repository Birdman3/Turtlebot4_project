# User PC Setup — Ubuntu/WSL

이 문서는 [공식 Discovery Server 문서](https://turtlebot.github.io/turtlebot4-user-manual/setup/discovery_server.html)의 **User PC Setup → Humble** 절차를 정리합니다.

## 1. 실행 위치와 사전 조건

- 로봇 서비스 복구는 완료된 상태입니다.
- 아래 작업은 **Windows PC의 Ubuntu/WSL 로컬 터미널**에서 수행합니다.
- 로봇 SSH 터미널에서는 이 PC 설정을 실행하지 않습니다.
- PC에도 ROS 2 Humble 및 필요한 DDS 환경이 준비돼 있어야 합니다. 이 대화에서는 PC의 설치 상태까지 확인하지 않았습니다.
- 로봇과 PC가 서로 통신 가능한 네트워크에 있어야 합니다. WSL의 가상 네트워크·방화벽 영향은 별도 확인 대상입니다.
- Galactic, Humble, Jazzy 절차를 섞지 않습니다.

## 2. 공식 설정 명령

공식 문서의 원래 명령:

```bash
wget -qO - https://raw.githubusercontent.com/turtlebot/turtlebot4_setup/humble/turtlebot4_discovery/configure_discovery.sh | bash <(cat) </dev/tty
```

이번 사례에서는 출력 없이 진행되지 않았습니다. `wget -q`가 오류를 숨길 수 있으므로 다음과 같이 다운로드와 실행을 분리하면 진단하기 쉽습니다.

```bash
wget --timeout=15 --tries=1 -O /tmp/turtlebot4-configure-discovery.sh https://raw.githubusercontent.com/turtlebot/turtlebot4_setup/humble/turtlebot4_discovery/configure_discovery.sh
```

**다운로드가 성공한 경우에만** 실행합니다.

```bash
bash /tmp/turtlebot4-configure-discovery.sh
```

다운로드 오류가 나오면 실행을 반복하지 말고 [WSL DNS 진단](04-wsl-dns.md)으로 이동합니다. `wget -O`는 실패해도 빈 파일을 남길 수 있습니다.

## 3. 입력할 값

| 질문 | 값 |
|---|---|
| ROS_DOMAIN_ID | 0 |
| Discovery Server ID | 0 |
| Discovery Server IP | 현재 로봇의 Wi-Fi IP |
| Discovery Server Port | 11811 |
| 수정 / 추가 / 완료 | `d` (done) |

PC 자신의 IP 또는 `127.0.0.1`을 넣는 것이 아닙니다. 주소가 바뀌었으면 현재 로봇 주소를 사용합니다.

## 4. 스크립트가 자동 수행하는 작업

문서의 “Once you have entered these values, the script will perform the following”은 **값 입력 후 스크립트가 아래 작업을 자동으로 수행한다**는 의미입니다.

1. `/etc/turtlebot4_discovery/` 디렉터리 생성
2. 필요한 설정이 담긴 `setup.bash` 생성
3. `~/.bashrc`에 다음 줄 추가
   ```bash
   source /etc/turtlebot4_discovery/setup.bash
   ```
4. 이전 Discovery Server 방식에서 사용하던 IP 라우트가 있는지 확인하고 있으면 제거

사용자가 각 폴더와 파일을 수동 생성하라는 설명이 아닙니다. 기존 방식의 라우트를 정리한다는 의미이지, PC의 모든 라우팅을 삭제한다는 의미도 아닙니다.

## 5. 설정 적용 및 조회

스크립트가 성공적으로 완료된 후 같은 **PC WSL 터미널**에서:

```bash
source ~/.bashrc
ros2 daemon stop
ros2 daemon start
```

약 10초 뒤:

```bash
ros2 topic list
ros2 node list
```

처음에는 목록이 덜 보일 수 있으므로 잠시 후 다시 조회할 수 있습니다. PC에서 로봇 토픽이 나타나면 발견 기능이 확인된 것입니다. 실제 데이터 수신도 별도로 확인해야 합니다.

**이 기록에서는 PC 스크립트 다운로드부터 실패했으므로, 이 단계까지 완료됐다고 기록하지 않습니다.**

## daemon과 서버의 차이

| 명령 또는 서비스 | 역할 |
|---|---|
| `ros2 daemon start` | ROS CLI가 그래프 정보를 조회하는 데 사용하는 백그라운드 daemon 시작 |
| `fastdds discovery ...` | Fast DDS Discovery Server를 직접 실행 |
| 로봇의 `discovery.service` | Discovery Server의 서비스 실행 및 자동 시작 관리 |

PC 연결을 위해 PC에 별도 Discovery Server를 추가하는 절차가 아닙니다. 이미 실행 중인 로봇 서버의 주소를 PC에 알려주는 과정입니다.
