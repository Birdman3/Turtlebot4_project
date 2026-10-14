# WSL DNS 오류 — 해결 확인이 남은 항목

## 1. 관찰된 오류

사용자 PC의 Ubuntu/WSL에서 설정 스크립트 다운로드가 실패했습니다.

```text
Resolving raw.githubusercontent.com ... failed: Temporary failure in name resolution.
wget: unable to resolve host address 'raw.githubusercontent.com'
```

오류가 발생한 단계는 GitHub 호스트 이름을 IP로 바꾸는 DNS 조회입니다. 아직 HTTP 다운로드나 ROS 설정 단계까지 도달하지 못했습니다.

실패 후 빈 파일을 `bash`로 실행하면 아무 출력 없이 끝날 수 있습니다. 이를 설정 성공으로 판단하면 안 됩니다.

## 2. 진행한 확인과 결과

### WSL: 외부 IPv4 ping

```bash
ping -c 3 1.1.1.1
```

결과: 3회 전송, 수신 0, 100% 패킷 손실.

이것만으로 인터넷 전체가 끊겼다고 확정할 수 없습니다. ICMP가 차단됐을 수도 있기 때문입니다.

### WSL: 이름 조회

```bash
getent hosts raw.githubusercontent.com
```

결과: `2606:...` 형태의 IPv6 주소 네 개가 출력됐습니다.

**정정해서 이해할 점:** 이 결과는 해당 시점의 IPv6 이름 조회가 됐다는 뜻입니다. IPv4 DNS나 HTTPS 접속까지 정상임을 보장하지 않습니다. “DNS 전체가 해결됐다”고 단정할 수 없습니다.

### WSL: IPv4 다운로드 재시도

```bash
wget -4 --timeout=15 --tries=1 -O /tmp/turtlebot4-configure-discovery.sh https://raw.githubusercontent.com/turtlebot/turtlebot4_setup/humble/turtlebot4_discovery/configure_discovery.sh
```

결과: 동일한 이름 해석 오류가 계속 발생했습니다.

### Windows PowerShell: IPv4 DNS 조회

```powershell
Resolve-DnsName raw.githubusercontent.com -Type A
```

결과: A 레코드 네 개가 정상 출력됐습니다.

이것은 Windows의 해당 호스트 IPv4 이름 조회가 성공했다는 근거입니다. Windows의 모든 인터넷 통신이 정상이라는 증명은 아니지만, Windows와 WSL의 DNS 동작에 차이가 있음을 보여줍니다.

## 3. 유선에서 Wi-Fi로 바뀐 것이 원인인가?

사용자는 PC 연결이 유선에서 무선으로 바뀐 사실을 알려줬습니다.

- 무선 연결 자체가 문제라고 단정할 수 없습니다.
- 네트워크 전환 후 WSL의 DNS·가상 네트워크 상태가 갱신되지 않았을 가능성은 있습니다.
- 이 전환이 직접적인 원인이었는지는 아직 검증하지 않았습니다.
- 로봇에 직접 연결하는 Wi-Fi와 인터넷이 되는 집 공유기 Wi-Fi도 구분해야 합니다.

현재 증거상 우선 조사 대상은 **WSL의 IPv4 DNS 및 네트워크 경로**입니다.

## 4. 마지막으로 안내한 다음 조치

아래는 방법을 안내했으나 **실행 후 성공 화면을 아직 받지 못한 단계**입니다.

### A. Windows PowerShell에서 WSL 종료

WSL에서 작업 중인 파일을 저장한 후:

```powershell
wsl --shutdown
```

모든 실행 중인 WSL 배포판과 그 안의 프로그램이 종료됩니다. 로봇 자체를 종료하는 명령은 아닙니다. 이후 Ubuntu를 다시 엽니다.

[Microsoft WSL 문제 해결](https://learn.microsoft.com/en-us/windows/wsl/troubleshooting)

### B. 다시 연 WSL에서 다운로드

```bash
wget -4 --timeout=15 --tries=1 -O /tmp/turtlebot4-configure-discovery.sh https://raw.githubusercontent.com/turtlebot/turtlebot4_setup/humble/turtlebot4_discovery/configure_discovery.sh
```

성공한 경우에만:

```bash
bash /tmp/turtlebot4-configure-discovery.sh
```

이후 [PC 설정 문서](03-user-pc-setup.md)의 값 입력과 daemon 재시작을 진행합니다.

## 5. 그래도 실패할 때 수집할 정보

다음은 **추가 진단 제안이며 아직 실행 결과가 없는 명령**입니다.

Windows PowerShell:

```powershell
wsl --version
wsl --status
wsl -l -v
Resolve-DnsName raw.githubusercontent.com -Type A
```

Ubuntu/WSL:

```bash
ip route
cat /etc/resolv.conf
cat /etc/wsl.conf
getent ahostsv4 raw.githubusercontent.com
```

`/etc/wsl.conf`가 없을 수 있으며, 그 사실 자체가 오류는 아닙니다. WSL·Windows 버전과 현재 설정을 확인한 뒤 DNS tunneling 또는 네트워크 설정을 검토합니다. 기존 설정을 확인하지 않고 resolv.conf를 덮어쓰거나 방화벽을 끄는 조치는 이 기록에서 수행하지 않았습니다.

[Microsoft WSL 네트워크 및 DNS tunneling](https://learn.microsoft.com/en-us/windows/wsl/networking)

## 6. 완료 판정

- [x] Windows에서 해당 호스트의 A 레코드 조회 성공
- [ ] WSL 재시작 후 IPv4 DNS 조회 성공
- [ ] WSL에서 스크립트 다운로드 성공
- [ ] User PC Setup 스크립트 완료
- [ ] PC에서 로봇 토픽 및 노드 조회
- [ ] 필요한 토픽의 실제 데이터 수신

DNS 문제를 해결해도 WSL의 DDS 통신까지 자동으로 검증되는 것은 아닙니다. DNS와 ROS 통신 결과를 각각 기록합니다.
