# Oracle Database Engineering and Administration

Oracle 19c 기반 데이터베이스 운영, 모니터링, 백업/복구, RAC 고가용성 구성을 실습한 프로젝트입니다. 단순 설치 기록이 아니라 장애 상황을 진단하고 복구 절차를 설계하는 데이터베이스 엔지니어링 관점으로 정리했습니다.

## 프로젝트 목표

```text
Oracle Linux 환경 구성
-> Oracle 19c Single Instance 운영
-> 인스턴스 상태 모니터링
-> NOARCHIVELOG / ARCHIVELOG 복구 시나리오 실습
-> 2-Node RAC, ASM, TAF 구성
-> 장애 분석과 리소스 최적화
```

## 저장소 구성

```text
Oracle-Database-Engineering/
  README.md
  01_Environment_Setup/
    README.md
  02_Administration/
    README.md
  03_SQL_and_Monitoring/
    README.md
  04_Backup_and_Recovery/
    Scenario_NOARCHIVE.md
    Scenario_ARCHIVE/
      Scenario_2.md
  05_Oracle_RAC/
    README.md
```

## 주요 학습 주제

### 1. Environment Setup

- Oracle Linux 환경 변수 설정
- `ORACLE_BASE`, `ORACLE_HOME`, `ORACLE_SID` 이해
- SQL*Plus 관리자 접속 흐름 정리

### 2. Administration

- Oracle background process 확인
- startup 단계인 NOMOUNT, MOUNT, OPEN 구분
- shutdown 방식별 차이와 복구 영향 정리

### 3. SQL and Monitoring

- `v$instance`, `v$session`, `v$log` 기반 상태 진단
- 세션, 연결, redo log 상태 확인
- 백엔드 connection pool 이슈와 DB 세션 모니터링 연결

### 4. Backup and Recovery

- NOARCHIVELOG 모드에서 복구 가능한 범위와 한계 확인
- ARCHIVELOG 모드에서 온라인 복구와 완전 복구 흐름 실습
- data file, control file, redo log 유실 시나리오 정리

### 5. Oracle RAC

- 2-Node RAC 네트워크 구조 이해
- ASM 기반 공유 스토리지 구성
- VIP, SCAN, TAF를 활용한 Failover 구조 검증
- GI 설치 중 리소스 부족 문제를 로그 분석과 메모리 증설로 해결

## 포트폴리오 핵심 포인트

- Oracle 운영을 SQL뿐 아니라 OS, 메모리, 스토리지, 네트워크까지 함께 보는 관점으로 정리했습니다.
- 장애 발생 시 DB가 어느 startup 단계에서 멈추는지 기준으로 원인을 좁히는 방식을 학습했습니다.
- ARCHIVELOG 여부에 따라 복구 전략이 달라진다는 점을 시나리오별로 정리했습니다.
- RAC 설치 과정에서 발생한 리소스 병목을 `installActions.log` 분석으로 진단하고 인프라 사양을 재산정했습니다.

## 운영 관점 주의사항

- 복구 명령은 데이터 손실 가능성이 있으므로 백업본, archive log, redo log 상태를 먼저 확인해야 합니다.
- control file과 redo log는 운영 안정성을 위해 다중화가 필요합니다.
- RAC 환경은 단일 DB보다 메모리와 네트워크 요구사항이 크므로 설치 전 sizing이 중요합니다.
