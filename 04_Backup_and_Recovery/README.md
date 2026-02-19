## 🛠 4. 파일 유실 및 장애 대응 시나리오 (Hands-on Recovery)

Oracle Database 운영 중 발생할 수 있는 주요 파일 유실 상황을 시나리오별로 재현하고, **User Managed Recovery** 방식을 통해 데이터 무결성을 복구한 실습 기록입니다.

#### 📁 0. 사전 준비: 백업 대상 및 모드 확인
장애 복구의 시작은 현재 시스템의 구성과 백업 상태를 정확히 파악하는 것입니다. 
* **백업 대상 확인:** Datafiles, Controlfiles, Redologfiles의 경로를 파악합니다. 
* **로그 모드 점검:** `archive log list`를 통해 `NOARCHIVELOG` 혹은 `ARCHIVELOG` 여부를 확인합니다. 
* **일관성 있는 백업 (Cold Backup):** DB를 정상 종료(`shutdown immediate`)한 후 물리적 파일 복사(cp)를 통해 전체 백업을 수행했습니다. 

---

#### 🚨 Case 1. 특정 데이터 파일 손상 (백업 후 리두 정보 존재 시)
* **장애 상황:** `users01.dbf` 파일이 삭제되어 인스턴스가 `MOUNT` 단계에서 멈추는 현상 발생. 
* **해결 방법:** 가장 최근의 백업본을 복원(Restore)한 후, 리두 정보를 적용하여 **완전 복구(Complete Recovery)**를 수행합니다. 
* **실무 명령어:**
    ```sql
    -- 1. 손상된 파일을 offline drop으로 설정 (NOARCHIVELOG 시 필수)
    ALTER DATABASE DATAFILE 7 OFFLINE DROP; 
    -- 2. DB를 Open한 후 백업본 복사 (OS 레벨)
    !cp /backup/users01.dbf /u01/app/oracle/oradata/ORCL/ 
    -- 3. 리두 정보를 이용한 복구 및 온라인 전환
    RECOVER DATAFILE 7; 
    ALTER DATABASE DATAFILE 7 ONLINE; 
    ```

#### 🚨 Case 2. 백업본이 없는 신규 데이터 파일 손상 (리두 정보 존재 시)
* **장애 상황:** 백업을 받기 전 생성된 신규 테이블스페이스의 데이터 파일(`tbs01.dbf`)이 유실됨. 
* **해결 방법:** 물리적 파일이 없더라도 **Control File의 메타데이터와 리두 정보**를 이용해 빈 파일을 생성하고 복구합니다. 
* **실무 명령어:**
    ```sql
    -- 1. 유실된 파일에 대해 물리적인 빈 파일 생성
    ALTER DATABASE CREATE DATAFILE '/path/tbs01.dbf'; 
    -- 2. 리두 정보를 적용하여 데이터 복구
    ALTER DATABASE RECOVER DATAFILE '/path/tbs01.dbf'; 
    -- 3. 정상 상태(ONLINE)로 복귀
    ALTER DATABASE DATAFILE '/path/tbs01.dbf' ONLINE; 
    ```

#### 🚨 Case 3. 시스템(SYSTEM) 데이터 파일 손상
* **장애 상황:** DB 가동의 핵심인 `system01.dbf` 유실. 가동 중인 세션이 끊기고 재기동 시 `OPEN`이 불가능함. 
* **해결 방법:** SYSTEM 파일은 `OFFLINE` 전환이 불가능하므로, 반드시 **MOUNT 단계**에서 복구 작업을 진행해야 합니다. 
* **실무 명령어:**
    ```sql
    -- 1. 백업본으로부터 SYSTEM 데이터 파일 복원
    !cp system01.dbf /u01/app/oracle/oradata/ORCL/ 
    -- 2. MOUNT 상태에서 테이블스페이스 단위 복구 수행
    RECOVER TABLESPACE SYSTEM; 
    -- 3. 데이터베이스 오픈
    ALTER DATABASE OPEN; 
    ```

#### 🚨 Case 4. 완전 복구가 불가능한 경우 (NOARCHIVELOG 모드 리두 유실)
* **장애 상황:** 로그 스위치가 발생하여 필요한 리두 정보가 덮어씌워진(Overwrite) 경우. 
* **해결 방법:** **불완전 복구**를 수행해야 하며, 컨트롤/데이터/리두 로그 파일을 **전부 마지막 백업 시점으로 복원**하여 과거로 되돌립니다. 
* **실무 명령어:**
    ```bash
    # 1. 인스턴스 강제 종료 후 모든 백업본 복사 (OS 레벨)
    shutdown abort; 
    cp /backup/noarch/* /u01/app/oracle/oradata/ORCL/ 
    # 2. 마지막 백업 시점의 상태로 DB 기동
    startup; 
    ```

---

#### 💡 핵심 학습 포인트 (Lessons Learned)
1.  **SCN과 데이터 일관성:** 모든 파일의 SCN(System Change Number)이 일치해야 DB가 `OPEN`될 수 있음을 확인했습니다. 
2.  **로그 모드의 결정적 차이:** `ARCHIVELOG` 모드 없이는 장애 발생 시점까지의 완전 복구가 매우 제한적임을 실습을 통해 체감했습니다. 
3.  **핵심 파일의 특수성:** SYSTEM 이나 UNDO 파일은 일반 데이터 파일과 복구 메커니즘이 다르다는 점을 숙지했습니다. 
