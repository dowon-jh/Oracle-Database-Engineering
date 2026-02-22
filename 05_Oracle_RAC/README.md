## 🏗️ 5. Oracle 19c RAC (Real Application Clusters) 실습

고가용성(High Availability)과 성능 확장성(Scalability) 확보를 위해 2-Node RAC 환경을 구축하며 클러스터웨어 설치 및 DB 인스턴스 구성 전 과정을 학습했습니다.

### 1) 2-Node RAC 네트워크 및 아키텍처 설계
RAC의 핵심인 '공유 저장소(Shared Everything)' 구조와 노드 간 통신을 위한 네트워크 설정을 직접 수행했습니다.
- **Public IP:** 외부 사용자 접속용
- **Private IP (Interconnect):** 노드 간 데이터 전송(Cache Fusion) 및 Heartbeat 체크용 전용망
- **Virtual IP (VIP) & SCAN:** 특정 노드 장애 시 클라이언트 접속 유지(Failover) 및 부하 분산을 위한 가상 IP 구성

### 2) ASM (Automatic Storage Management) 및 디스크 그룹 구성
공유 스토리지의 효율적인 관리와 데이터 보호를 위해 오라클 전용 볼륨 매니저인 ASM을 활용했습니다.

* **ASM의 주요 역할:**
    - **공유 스토리지 관리:** 여러 노드가 동시에 동일한 데이터 파일에 접근할 수 있는 환경 제공.
    - **데이터 스트라이핑:** 데이터를 디스크 그룹에 골고루 분산 저장하여 I/O 성능 최적화.
    - **미러링:** 디스크 장애 시에도 데이터 유실을 방지하여 데이터베이스 가용성 보장.
* **실습 디스크 그룹 구성:**
    - `+DATA`: 실제 데이터베이스의 데이터 파일, 인덱스, 리두 로그 등을 저장하는 메인 영역.
    - `+ASM`: 클러스터 유지에 필수적인 OCR(Oracle Cluster Registry) 및 Voting Disk 정보를 저장하여 클러스터 무결성 관리.

### 3) 실전 트러블슈팅: 리소스 병목 분석 및 인프라 최적화
설치 과정에서 발생한 기술적 한계를 분석하고 해결한 경험은 인프라 사이징의 중요성을 깨닫게 했습니다.

- **문제 상황:** 노드당 16GB 메모리 환경에서 GI 설치 스크립트(`runInstaller`) 실행 중 세션 중단 및 시스템 프리징 현상 발생.
- **원인 분석 및 사고 방식:** 1. `installActions.log` 및 OS 로그 분석을 통해 설치 프로세스가 요구하는 JVM Heap 메모리와 오라클 인스턴스의 최소 점유 메모리가 가용 메모리를 초과함을 파악.
  2. RAC 환경은 각 노드가 독자적인 메모리뿐만 아니라 노드 간 동기화를 위한 고정 메모리(Fixed Size) 비중이 높다는 점에 주목.
- **해결 방안:** 각 노드의 메모리를 **32GB로 증량**하여 안정적인 SGA 할당 및 OS 가용 공간 확보.
- **결과:** 노드 간 Interconnect 통신 지연 해소 및 안정적인 2-Node RAC 클러스터 구축 성공.

### 4) 주요 관리 및 검증 명령어
- **`crsctl check crs`**: Clusterware 전체 서비스 상태 확인
- **`crsctl stat res -t`**: 모든 리소스(VIP, SCAN, ASM, DB)의 온라인 상태 모니터링
- **`cluvfy (Cluster Verification Utility)`**: 설치 전후 네트워크 및 스토리지 정합성 검증

#### 💡 학습 포인트
- **인프라 사이징의 중요성:** 단순 설치를 넘어 소프트웨어가 요구하는 리소스에 대한 정확한 산정과 최적화가 서비스 안정성에 직결됨을 경험했습니다.
- **스토리지 가상화 이해:** ASM을 통해 복잡한 공유 스토리지 환경을 단일 파일 시스템처럼 관리하며, 고가용성을 하드웨어와 소프트웨어 양면에서 확보하는 법을 체득했습니다.
---

### 5) 애플리케이션 서비스 분산 및 가동성 테스트 (Workload Management)
단순한 인스턴스 구동을 넘어, 업무 성격에 따른 서비스 분산 및 TAF(Transparent Application Failover) 설정을 통해 진정한 RAC의 고가용성을 검증했습니다.

* **서비스 기반 부하 분산:** `srvctl` 명령어를 통해 업무별 전용 서비스(ERP, PMS)를 생성하고 각 노드에 배치했습니다.
* **검증 로그 (`crsctl stat res -t`):**
    ```bash
    ora.orcl.erp.svc            ONLINE  ONLINE   oel7rac1  -- ERP 업무 1번 노드 할당
    ora.orcl.pms.svc            ONLINE  ONLINE   oel7rac2  -- PMS 업무 2번 노드 할당
    ora.orcl.pms_preconnect.svc ONLINE  ONLINE   oel7rac1  -- 장애 대비 Pre-connect 설정
    ```
* **실습 내용:** - 특정 노드 장애 시 서비스가 생존 노드로 자동 이동(Failover)하는 메커니즘 확인.
    - 클라이언트 접속 시 `TNSNAMES.ORA` 설정을 통해 장애 발생 시에도 사용자 세션이 끊기지 않고 유지되는 것을 확인.



---

#### 💡 학습 포인트
- **서비스 가용성:** 인스턴스가 살아있어도 서비스가 죽으면 사용자 접속이 불가능함을 이해하고, 서비스 단위의 상태 모니터링 중요성을 학습했습니다.
- **Failover 전략:** `preconnect` 설정을 통해 장애 시 재연결 시간을 최소화하는 고급 TAF 옵션을 실무적으로 적용해 보았습니다.
