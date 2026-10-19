# 강남점 네트워크 장애 진단 기록

> 2026년 9월 GNS3 교육 실습 및 당시 작성한 트러블슈팅 매뉴얼을 바탕으로 재정리했습니다. **실제 장비에서 현재 재현·재검증한 로그는 아닙니다.**

## 1. 장애 시 확인 순서

```text
PC 주소/마스크/DHCP 사용 여부
   ↓
자기 VLAN Gateway (SVI)
   ↓
Access VLAN/Trunk · DIST의 Inter-VLAN Routing
   ↓
DIST ↔ Core L3 Transit · OSPF Route
   ↓
Core ↔ Server-SW ↔ DHCP 서버
   ↓
DHCP Relay / 서비스 상태 / 클라이언트 할당 결과
   ↓
본사 연결 Transit · 팀 Address Plan · WAN OSPF
```

한 번에 전체 설정을 바꾸지 않고 **처음 실패하는 구간**을 찾는 것을 기준으로 했습니다.

## 2. 사례별 증상 → 확인 → 조치

| 사례 | 증상 | 확인·조치 | 확인 결과/상태 |
|---|---|---|---|
| VLAN 간 통신 실패 | 자기 Gateway Ping 성공, 타 VLAN PC Ping 실패 | VLAN/SVI 확인 후 DIST에서 `ip routing` 활성화 | 매뉴얼에 통신 정상화 절차 기록 |
| 목적지 PC Ping 실패 | 중간 장비/게이트웨이는 정상, Windows PC가 ICMP에 응답하지 않음 | Windows Defender Firewall의 ICMP 차단 여부 확인 | 테스트 시 호스트 방화벽 정책 확인 |
| DHCP 주소 할당 실패 | 서버 `10.10.20.10`에 Ping 되지만 클라이언트가 IP를 못 받음 | 라우팅·`ip helper-address`·`dhcpd` 확인 후 실습 IOS의 `service dhcp` 활성화 | VLAN 30·70 할당 성공이 매뉴얼에 기록됨 |
| APIPA (`169.254.x.x`) | 클라이언트에 정상 대역 IP와 게이트웨이가 없음 | `ipconfig /all`, PC → Gateway → DIST → Core → 서버 경로 점검 | DHCP 응답 미수신을 구분하는 신호로 사용 |
| DHCP 수동 설정 충돌 | `ipconfig /release` 사용 불가 | `ipconfig /all`에서 DHCP 사용 여부 확인 후 자동 할당으로 전환 | DHCP 테스트 조건 정상화 |
| Transit IP 중복 | 본사와 강남 연동 시 초기 주소 계획 충돌 | 팀 계획 통합 검토 후 Core–HQ `192.168.40.12/30` 사용 | 매뉴얼의 변경 이력 확인 |
| OSPF Router ID 경고 | Ping은 성공하지만 `DUP_RTRID_AREA`, `FLOOD_WAR` 등 경고 | `show ip ospf`로 각 Router ID 및 중복 여부 점검 | **최종 경고 제거 여부는 현재 확보 자료만으로 확인 불가** |

## 3. DHCP 장애를 어떻게 점검했나

### 구성

- DHCP 서버: Rocky Linux 9 VM, `10.10.20.10/24`
- 서버 게이트웨이: `10.10.20.1`
- 사용자 VLAN SVI: `ip helper-address 10.10.20.10`으로 DHCP 요청 중계
- 서버 설정 경로: `/etc/dhcp/dhcpd.conf`

### 점검용 명령어

Cisco DIST에서:

```cisco
show ip interface brief
show ip route
! SVI 설정에서 ip helper-address 값을 확인
! DHCP/BOOTP Relay 관련 service dhcp 상태 확인
```

Rocky Linux에서:

```bash
dhcpd -t -cf /etc/dhcp/dhcpd.conf
systemctl status dhcpd
```

Windows PC에서:

```powershell
ipconfig /all
ipconfig /renew
```

문제 당시에는 DHCP 서버까지의 Ping과 Relay 설정을 확인했음에도 할당되지 않았습니다. **강사님의 기술 자문을 받아 실습 Cisco IOS의 `service dhcp`를 활성화한 뒤** VLAN 30·70에서 자동 할당이 성공한 것으로 매뉴얼에 기록되어 있습니다.

> `service dhcp`가 모든 Cisco 장비에서 반드시 추가로 필요한 명령이라는 뜻은 아닙니다. 당시 실습 환경에서의 해결 과정입니다.

## 4. 핵심 학습

- Ping 성공은 **해당 IP 경로의 도달성**을 보여줄 뿐 DHCP·OSPF가 정상 동작한다는 증거는 아닙니다.
- SVI 구성만으로 VLAN 간 통신이 자동 활성화되지는 않으며 L3 라우팅도 확인해야 합니다.
- 팀 프로젝트에서는 지점별 IP 대역뿐 아니라 **서버망·Transit·WAN까지 통합 주소 계획**이 필요합니다.
- 동작 확인은 설정 명령 자체보다 **클라이언트 IP, OSPF 경로, 실제 Ping 등 결과**를 기준으로 해야 합니다.