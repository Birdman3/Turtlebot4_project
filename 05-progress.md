# 진행 기록과 남은 확인 항목

## 시간순 기록

날짜는 대화 기준이며, 기기 로그의 시스템 시간과 혼동하지 않습니다.

| 순서 | 상황 / 조치 | 결과 |
|---|---|---|
| 1 | 로봇 SSH에서 `ros2 topic list` | 기본 토픽 두 개만 보임 |
| 2 | 환경변수와 `ros2 node list` 확인 | Humble / Fast DDS / domain 0, 노드 목록 비어 있음 |
| 3 | 로봇 서비스 확인 | 서비스와 여러 노드 프로세스 실행 중 |
| 4 | UDP 11811 소켓 확인 | 출력 없음 |
| 5 | 서비스 목록과 setup.bash 확인 | 서버 주소 설정은 있으나 discovery.service는 보이지 않음 |
| 6 | `fastdds discovery -i 0 -p 11811` 수동 실행 | 서버 소켓 확인 |
| 7 | daemon 재시작만 시도 | 토픽 미복구 |
| 8 | 로봇 서비스 재시작 + 환경 로드 + daemon 재시작 | 토픽과 노드 목록 복구 |
| 9 | 설정 도구 Discovery 메뉴 확인 | Enabled=True, port=11811, ID=0 |
| 10 | 시작 스크립트 및 공식 설치 코드 확인 | 별도 discovery.service 등록 구조 확인 |
| 11 | Robot Upstart Install / Start 복구 절차 안내 후 확인 | 토픽 정상, discovery.service active / enabled 확인 |
| 12 | User PC Setup 문서 해석 및 PC 설정 안내 | 실행 장소가 PC Ubuntu/WSL임을 구분 |
| 13 | WSL에서 설정 스크립트 다운로드 | DNS 오류 |
| 14 | WSL ping / getent / IPv4 wget 확인 | ping 무응답, IPv6 조회 성공, IPv4 wget 이름 조회 실패 |
| 15 | Windows PowerShell DNS 확인 | IPv4 A 레코드 조회 성공 |
| 16 | WSL 종료·재실행 후 재시도 안내 | 실행 후 결과 미확인 |

## 복구 후 확인된 토픽 목록

아래는 사용자 제공 화면에서 확인한 목록입니다. 모든 TurtleBot 4의 필수 목록이나 센서 데이터의 정상 수신을 뜻하지 않습니다.

```text
/battery_state
/cmd_vel
/diagnostics
/diagnostics_agg
/diagnostics_toplevel_state
/dock_status
/function_calls
/hazard_detection
/hmi/buttons
/hmi/display
/hmi/display/message
/hmi/led
/imu
/interface_buttons
/ip
/joint_states
/joy
/joy/set_feedback
/mouse
/oakd/rgb/preview/image_raw
/parameter_events
/robot_description
/rosout
/scan
/tf
/tf_static
/wheel_status
```

## 복구 후 확인된 주요 노드

```text
/analyzers
/joint_state_publisher
/joy_linux_node
/oakd
/oakd_container
/robot_state_publisher
/rplidar_composition
/teleop_twist_joy_node
/turtlebot4_base_node
/turtlebot4_diagnostics
/turtlebot4_node
```

화면에 실행별로 이름이 바뀔 수 있는 `/launch_ros_...` 노드도 있었습니다.

## 확정하지 않은 사항

- 왜 discovery.service 등록이 처음에 없었는지에 대한 최초 원인
- 복구 후 로봇 재부팅 테스트
- OAK-D 실제 이미지 발행 정상 여부
- LiDAR 및 IMU 등의 실제 데이터 수신 상태
- WSL 버전과 PC ROS 설치 상태
- 유선 → 무선 전환이 DNS 오류의 직접적인 원인인지 여부
- WSL 재시작 후 DNS·다운로드 복구 여부
- PC에서 로봇 토픽 발견 및 데이터 수신 여부

## 다음 작업 순서

1. Windows PowerShell에서 `wsl --shutdown`, Ubuntu 재실행 후 다운로드 확인
2. 다운로드 성공 시 User PC Setup 실행 및 값 입력
3. PC 환경 적용과 daemon 재시작 후 토픽·노드 조회
4. 필요한 토픽의 데이터 수신 검증
5. 로봇을 재부팅할 수 있는 시점에 discovery.service와 토픽 재확인

새 결과가 확인되면 README의 상태 표와 이 문서의 체크 항목을 함께 갱신합니다.
