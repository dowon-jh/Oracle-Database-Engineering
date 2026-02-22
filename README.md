# 🏛️ Oracle Database Engineering & Administration

Oracle 19c 기반의 데이터베이스 엔지니어링 실무 역량을 쌓기 위한 종합 프로젝트 레포지토리입니다. 단순 설치를 넘어 대규모 데이터 환경을 위한 **고가용성(HA)** 구축과 **장애 대응(Backup & Recovery)** 시나리오 실습을 중점적으로 다루었습니다.

---

## 📂 Repository Structure

### [01. Environment Setup](./01_Environment_Setup)
- Oracle Linux 환경에서의 19c Single & RAC 설치 가이드
- 인프라 리소스 분석 및 커널 파라미터 최적화

### [02. Administration](./02_Administration)
- DB 초기화 파라미터(SPFILE/PFILE) 및 컨트롤 파일 관리
- 유저 권한 제어 및 테이블스페이스 설계

### [03. SQL & Monitoring](./03_SQL_and_Monitoring)
- 효율적인 데이터 조회를 위한 SQL 튜닝 및 성능 모니터링
- 인스턴스 상태 및 로그 분석

### [04. Backup & Recovery](./04_Backup_and_Recovery)
- **User Managed Backup** 기반의 24가지 장애 대응 시나리오 실습
- **NOARCHIVELOG vs ARCHIVELOG** 모드별 복구 전략 차별화
- `CREATE DATAFILE`을 이용한 무백업 파일 복구 및 불완전 복구(PITR) 수행

### [05. Oracle RAC (Real Application Clusters)](./05_Oracle_RAC)
- **2-Node RAC** 인프라 구축 및 클러스터웨어(GI) 관리
- ASM(Automatic Storage Management)을 이용한 공유 스토리지 최적화
- 서비스 기반 부하 분산 및 **TAF(Transparent Application Failover)** 검증

---

## 🛠️ Core Engineering Skills
* **Architecture:** Oracle Single/RAC 인스턴스 구조 및 메모리(SGA/PGA) 아키텍처 이해
* **Storage:** ASM(Automatic Storage Management) 활용 및 가상화 이해
* **Recovery:** 다양한 파일 유실 상황에 대한 논리적 진단 및 복구 프로세스 수립
* **Troubleshooting:** 설치 프로세스 병목 구간 분석 및 리소스 사이징(CPU/MEM) 최적화

---

## 💡 핵심 경험: "장애 분석 및 리소스 최적화를 통한 시스템 안정화"
RAC 설치 과정에서 발생한 메모리 부족 이슈를 `installActions.log` 분석을 통해 진단하고, 노드당 32GB 증량 및 커널 튜닝을 통해 안정적인 클러스터를 구축한 경험이 있습니다. 기술적 한계를 분석하고 인프라 사양을 재산정하는 엔지니어링 사고방식을 학습했습니다.
