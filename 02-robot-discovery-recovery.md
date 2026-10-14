# 로봇 토픽 두 개 문제와 Discovery Server 복구

## 1. 증상

로봇에 SSH로 접속한 터미널에서 다음 두 토픽만 보였습니다.

```bash
ros2 topic list
```

```text
/parameter_events
/rosout
```

`ros2 node list`는 빈 결과였습니다. 이 두 기본 토픽만으로 로봇 노드와 통신한다고 판단할 수 없습니다.

## 2. 실행 상태와 설정 확인

아래 명령은 **로봇 SSH 터미널**에서 실행했습니다.

```bash
printenv | grep -E '^(ROS_|RMW_|FASTRTPS_|FASTDDS_|CYCLONEDDS_)'
systemctl status turtlebot4.service --no-pager -l
sudo ss -lunp | grep ':11811'
sudo journalctl -u turtlebot4.service -b -n 80 --no-pager
```

관찰 결과:

- `turtlebot4.service`는 `active (running)`이었습니다.
- TurtleBot, LiDAR, 카메라 컨테이너, robot_state_publisher 등의 프로세스가 실행 중이었습니다.
- `ROS_DISCOVERY_SERVER=127.0.0.1:11811;`로 설정돼 있었습니다.
- `ss`의 11811 포트 필터는 출력 없이 끝났습니다. 명령이 멈춘 것이 아니라 일치하는 UDP 소켓이 없었습니다.
- 로그에 OAK-D의 `Cannot find any device with given deviceInfo` 오류가 반복됐습니다. 이 오류만으로 모든 노드가 안 보이는 현상을 설명할 수는 없었습니다.

추가 확인:

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'turtlebot|discovery|fastdds'
grep -nE 'DISCOVERY|ROS_DOMAIN_ID|RMW_IMPLEMENTATION|ROS_SUPER_CLIENT' /etc/turtlebot4/setup.bash
```

서비스 목록에는 `turtlebot4.service`만 확인됐고, `discovery.service`는 표시되지 않았습니다.

**판단:** 로봇 노드 프로세스는 떠 있으나, 터미널과 노드가 사용할 Discovery Server의 실행 또는 등록에 문제가 있을 가능성이 높았습니다.

## 3. 임시 서버로 원인 확인

**로봇 SSH 터미널 A**에서 실행:

```bash
fastdds discovery -i 0 -p 11811
```

이 명령은 서버를 계속 실행하므로 프롬프트가 돌아오지 않는 것이 정상입니다. 확인 중에는 터미널을 열어 둡니다. [Fast DDS CLI 문서](https://fast-dds.docs.eprosima.com/en/2.x/fastddscli/cli/cli.html)

**로봇 SSH 터미널 B**에서 확인:

```bash
sudo ss -lunp | grep ':11811'
```

`0.0.0.0:11811`과 `fast-discovery-` 프로세스가 확인됐습니다. UDP 소켓의 `UNCONN` 표시는 연결 오류를 뜻하지 않습니다.

이 상태에서 daemon만 재시작했을 때에는 토픽이 즉시 복구되지 않았습니다.

## 4. 로봇 서비스 재시작으로 토픽 복구

로봇을 정지한 상태로 두고 **터미널 B**에서 실행했습니다. 서비스 재시작 중 로봇 기능은 잠시 중단됩니다.

```bash
sudo systemctl restart turtlebot4.service
source /etc/turtlebot4/setup.bash
ros2 daemon stop
ros2 daemon start
```

약 15초 후:

```bash
ros2 topic list
ros2 node list
```

여러 토픽과 노드가 다시 나타났습니다. 서버 실행과 서비스/daemon 재시작을 함께 수행했으므로, 각각의 기여를 완전히 분리해 증명한 것은 아닙니다. 다만 서버 부재가 주요 원인이었다는 강한 근거입니다.

## 5. 자동 시작 복구

```bash
turtlebot4-setup
```

`ROS Setup → Discovery Server` 화면:

| 항목 | 값 |
|---|---|
| Enabled | True |
| Onboard Server - Port | 11811 |
| Onboard Server - Server ID | 0 |
| Offboard Server - IP | 빈 값 |
| Offboard Server - Port | 11811 |
| Offboard Server - Server ID | 1 |

Offboard IP가 비어 있으면 해당 Offboard 설정은 사용하지 않습니다. 이 사례에서는 기존 값이 이미 맞았습니다.

확인한 파일:

```bash
cat /etc/turtlebot4/discovery.sh
cat /usr/sbin/turtlebot4-start
```

`discovery.sh`의 핵심 내용:

```bash
#!/bin/bash
source /opt/ros/humble/setup.bash
fastdds discovery -i 0 -p 11811
```

파일 끝에 개행이 없으면 다음 셸 프롬프트가 같은 줄에 붙어 보일 수 있습니다. 이것만으로 파일 내용이 손상됐다고 판단하지 않습니다.

공식 Humble 설치 코드는 서버를 별도의 `discovery.service`로 설치합니다. 따라서 `turtlebot4-start`에 서버 명령이 직접 없다는 것 자체는 오류가 아닙니다. Discovery 설정이 켜져 있으면 Robot Upstart 설치 과정에서 서비스, 실행 스크립트, 부팅 시 실행 링크를 생성합니다. [공식 설치 코드](https://github.com/turtlebot/turtlebot4_setup/blob/humble/turtlebot4_setup/ros_setup.py)

복구 절차:

1. 임시 서버가 실행 중인 **터미널 A에서 Ctrl+C**로 종료합니다. 같은 포트의 중복 실행을 피합니다.
2. **터미널 B에서 `turtlebot4-setup`**을 실행합니다.
3. `ROS Setup → Robot Upstart → Install`을 선택합니다.
4. 완료 후 같은 메뉴의 `Start`를 선택합니다.
5. 설정 도구를 종료합니다.

여기서 Install은 현재 설정으로 로봇 자동 시작 서비스를 재설치하는 기능입니다. ROS 전체를 새로 설치하는 작업은 아닙니다.

## 6. 확인된 최종 결과

```bash
systemctl is-active discovery.service
systemctl is-enabled discovery.service
```

```text
active
enabled
```

토픽 조회도 정상적으로 복구됐습니다.

```bash
ros2 daemon stop
ros2 daemon start
ros2 topic list
ros2 node list
```

필요하면 daemon 시작 후 잠시 기다렸다가 목록을 다시 조회합니다.

- `active`: 현재 서비스 실행 중
- `enabled`: 부팅 시 자동 실행하도록 등록됨
- 재부팅 후에도 실제로 정상 실행되는지는 별도 확인이 필요함

목록에 나타난 주요 토픽은 `/scan`, `/imu`, `/battery_state`, `/cmd_vel`, `/tf`, `/tf_static`, `/oakd/rgb/preview/image_raw` 등입니다. 목록 복구만으로 각 센서 데이터가 실제로 계속 발행되는 것까지 검증한 것은 아닙니다.

이후에는 임시 `fastdds discovery` 명령을 매번 실행할 필요가 없습니다.
