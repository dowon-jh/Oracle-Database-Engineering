# 05. Oracle RAC

Oracle 19c RAC(Real Application Clusters)를 2-Node 환경으로 구성하며 고가용성, 공유 스토리지, 서비스 Failover를 실습한 문서입니다.

## 목표

- 2-Node RAC 네트워크 구조 이해
- Grid Infrastructure와 ASM 구성
- RAC 인스턴스 상태 점검
- 서비스 기반 부하 분산과 TAF 검증
- 설치 중 리소스 병목 원인 분석

## 1. RAC 네트워크 구조

RAC는 여러 노드가 하나의 데이터베이스를 공유하는 구조입니다. 이를 위해 접속용 네트워크와 노드 간 통신 네트워크를 분리합니다.

| 구분 | 역할 |
| --- | --- |
| Public IP | 외부 사용자 접속 |
| Private IP | Interconnect, Cache Fusion, Heartbeat |
| VIP | 노드 장애 시 빠른 접속 전환 |
| SCAN | 클라이언트 접속 분산과 단일 접속 주소 제공 |

## 2. ASM 구성

ASM(Automatic Storage Management)은 Oracle 전용 스토리지 관리 계층입니다.

주요 역할:

- 여러 노드가 공유하는 디스크 그룹 관리
- 데이터 스트라이핑으로 I/O 분산
- 미러링을 통한 장애 대응
- OCR, Voting Disk 등 클러스터 핵심 파일 관리

실습 디스크 그룹:

- `+DATA`: data file, index, redo log 저장
- `+ASM`: OCR, Voting Disk 등 클러스터 구성 정보 저장

## 3. 리소스 병목 트러블슈팅

문제:

- 노드당 16GB 메모리 환경에서 GI 설치 중 세션 중단과 시스템 프리징 발생

분석:

- `installActions.log`와 OS 로그를 확인했습니다.
- Grid Infrastructure 설치 과정의 JVM Heap 요구량과 Oracle 인스턴스 최소 메모리 요구량이 가용 메모리를 초과한 것으로 판단했습니다.
- RAC는 노드 간 동기화와 ASM 인스턴스 때문에 단일 인스턴스보다 메모리 여유가 더 필요합니다.

해결:

- 노드당 메모리를 32GB로 증량했습니다.
- 커널 파라미터와 가용 메모리를 재점검했습니다.
- Interconnect 통신과 ASM 인스턴스 기동을 다시 검증했습니다.

## 4. 관리 및 검증 명령

```bash
crsctl check crs
crsctl stat res -t
cluvfy
```

확인 대상:

- Clusterware 상태
- VIP, SCAN, ASM, DB resource online 여부
- 노드 간 네트워크와 공유 스토리지 정합성

## 5. 서비스 분산과 TAF

업무별 service를 생성해 노드별로 분산하고, 장애 시 다른 노드로 이동하는 Failover를 검증했습니다.

예시 구조:

```text
ERP service -> node 1
PMS service -> node 2
PMS preconnect service -> node 1
```

학습 포인트:

- RAC에서 인스턴스가 살아 있어도 service가 비정상이면 사용자는 접속할 수 없습니다.
- VIP, SCAN, TAF 설정은 실제 사용자 접속 유지와 직접 연결됩니다.
- 고가용성은 DB 인스턴스뿐 아니라 네트워크, 스토리지, service 단위로 검증해야 합니다.
