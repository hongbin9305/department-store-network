# 검증 자료 현황 및 재확인 체크리스트

포트폴리오에 적은 작업을 **확인된 캡처**, **실습 당시 문서에 기록된 결과**, **추후 재검증 필요 사항**으로 구분합니다. 현재 저장소만으로 모든 장비가 재실행 가능하거나 정상 운영 중이라고 주장하지 않습니다.

## 1. 확인된 자료

| 검증 항목 | 현재 근거 | 증빙 수준 |
|---|---|---|
| 강남점 계층형 토폴로지 | 저장소의 `gangnam-topology.png` 및 보관된 `webserver_PRAC.gns3`의 장비·링크 메타데이터 | 구조 확인 |
| OSPF 학습 경로 | 저장소 `ospf-routing.png`의 Core `show ip route` 출력 | 캡처 확인 |
| VLAN/IP 계획 | 실습 트러블슈팅 매뉴얼의 최종 Address Plan | 설정 기록 |
| DHCP Relay/서버 설정 | `ip helper-address`, 서버 `dhcpd.conf`의 scope 기록 | 설정 기록 |
| DHCP 자동 IP 할당 | `evidence/dhcp-vlan30-2026-10-19.png`의 VLAN 30 `ipconfig /all`, 당시 VLAN 30·70 매뉴얼 | VLAN 30 실제 캡처 확인 / VLAN 70은 당시 기록 |
| Core–HQ Transit 변경 | `192.168.40.12/30` 변경 이력 | 문서 기록 |

## 2. 프로젝트 재실행 시 확인할 항목

> 아래는 **이미 완료했다고 주장하는 목록이 아니라** 추후 프로젝트 원본과 실행 환경이 확보되면 재검증할 순서입니다.

- [ ] Cisco IOS 이미지, VirtualBox VM, 저장된 Cisco startup-config/NVRAM 등 필수 실행 환경 확인
- [ ] `show vlan-switch`에서 의도한 Access VLAN과 포트 소속 확인
- [ ] `show interfaces trunk`로 DIST 업링크 Trunk 확인
- [ ] `show ip interface brief`로 SVI 및 L3 링크 up/up 확인
- [ ] 게이트웨이, Inter-VLAN PC, DIST–Core Transit Ping
- [ ] `show ip ospf neighbor` FULL 및 `show ip route`의 OSPF 학습 경로 확인
- [ ] Rocky Linux `dhcpd -t`, `systemctl status dhcpd` 확인
- [x] **VLAN 30** 테스트 클라이언트의 `ipconfig /all`에서 `10.10.9.10/26`, 게이트웨이 `10.10.9.1`, DHCP 서버 `10.10.20.10` 및 임대 확인 — 2026-10-19 캡처 추가
- [ ] **나머지 VLAN**의 DHCP 임대 및 전체 통신 경로는 별도 검증
- [ ] Core–HQ Transit 주소 중복 및 WAN 연결 확인
- [ ] `DUP_RTRID_AREA`/`FLOOD_WAR` 관련 Router ID 중복 경고 제거 여부 재확인

## 3. 자료의 한계

- `.gns3` 파일에는 노드·링크 등 토폴로지 메타데이터가 있지만, **실행 가능한 전체 VM 이미지·Cisco 장비 설정이 모두 포함되었다고 볼 수 없습니다.**
- **VLAN 30**은 2026-10-19 새로 확보한 클라이언트 캡처로 DHCP 임대를 확인했습니다. 다른 VLAN의 현재 주소 할당과 전체 기능은 재검증하지 않았습니다.
- OSPF 라우팅 경로 캡처는 존재하지만 **전 구간의 장애 없는 상용 서비스 운영을 의미하지 않습니다.**
- 이 프로젝트는 **교육용 가상환경**에서 수행했습니다.