## 🛡️ NOARCHIVELOG 모드 장애 대응 핵심 시나리오

운영 환경의 제약(Archivelog 부재) 상황에서 데이터베이스의 일관성을 유지하고, 메타데이터를 재건하며, 로그 파일 유실에 대응하는 3가지 핵심 실습 기록입니다.

---

### 🚨 Case 1. 특정한 데이터 파일 손상 (백업 이후 리두 정보가 존재할 때)
> **포인트:** NOARCHIVELOG 모드에서도 리두 로그가 덮어씌워지기 전이라면 '완전 복구'가 가능함을 증명합니다.

**1) 장애 상황 및 진단**
* `users01.dbf` 파일 삭제 후 DB 기동 시 `ORA-01157` 에러 발생 및 `MOUNT` 상태 정지.
* `v$recover_file` 확인 시 7번 파일(Users) 유실 확인.

**2) 복구 절차 (Recovery)**
```sql
-- 1. NOARCHIVE 모드에서는 offline drop으로 처리
ALTER DATABASE DATAFILE 7 OFFLINE DROP;
ALTER DATABASE OPEN;

-- 2. 최신 백업본에서 유실된 파일 복사 (OS 레벨)
-- [oracle@oel7v9r1 noarch]$ cp users01.dbf /u01/app/oracle/oradata/ORCL/

-- 3. 백업 이후의 변경 정보(Redo)를 적용하여 복구
RECOVER DATAFILE 7;
ALTER DATABASE DATAFILE 7 ONLINE;
```
---

### 🚨 Case 14. Control File 유실 시 Trace 스크립트를 이용한 복구
> **포인트:** 바이너리 백업이 없는 최악의 상황에서 텍스트 형태의 Trace 파일을 이용해 컨트롤 파일을 수동 재생성하는 고급 기법입니다.

**1) 장애 상황 및 진단**

control01.ctl 삭제 후 shut abort 상황. 기동 시 ORA-00205 발생.

**2) 복구 절차 (Re-creating Controlfile)**
- 백업 컨트롤 파일을 이용해 MOUNT 한 뒤, 텍스트 스크립트(trace)를 추출합니다.
```sql
-- 1. 트레이스 파일 생성
ALTER DATABASE BACKUP CONTROLFILE TO TRACE AS '/home/oracle/cre_con.sql';

-- 2. NOMOUNT 단계에서 컨트롤 파일 재생성 스크립트 실행
CREATE CONTROLFILE REUSE DATABASE "ORCL" NORESETLOGS NOARCHIVELOG
    MAXLOGFILES 16
    MAXLOGMEMBERS 3
    MAXDATAFILES 100
    LOGFILE
      GROUP 1 '/u01/app/oracle/oradata/ORCL/redo01.log' SIZE 10M,
      GROUP 2 '/u01/app/oracle/oradata/ORCL/redo02.log' SIZE 10M
    DATAFILE
      '/u01/app/oracle/oradata/ORCL/system01.dbf',
      '/u01/app/oracle/oradata/ORCL/users01.dbf'
    CHARACTER SET AL32UTF8;
```

**3) 검증 및 사후 조치** 

- RECOVER DATABASE 후 OPEN 성공.

- 연결이 끊긴 TEMP 파일을 ALTER TABLESPACE TEMP ADD TEMPFILE ... REUSE; 명령어로 재연결.

---

### 🚨 Case 24.Current Redo Log File 삭제 시 복구
> **포인트:** DB의 심장인 '현재 기록 중인 로그'가 사라졌을 때, 로그 클리어(CLEAR LOGFILE) 명령어를 통한 복구 경로를 익힙니다.

**1) 장애 상황 및 진단**

- 상태가 CURRENT인 로그 그룹(Group 3) 강제 삭제 후 인스턴스 종료.

- 재기동 시 로그 파일 유실로 인해 OPEN 실패.

**2) 복구 절차 (Clear Logfile)**

- NOARCHIVELOG 모드에서는 아카이브 로그가 없으므로 해당 로그 그룹을 초기화하여 기동을 시도합니다.

```sql
-- 1. MOUNT 상태로 기동
STARTUP MOUNT;

-- 2. 유실된 Current 로그 그룹 초기화 (장애 시점 이후 데이터 유실 감수)
ALTER DATABASE CLEAR LOGFILE GROUP 3;

-- 3. 데이터베이스 오픈
ALTER DATABASE OPEN;
```
**3) 핵심 포인트**

- CURRENT 로그 유실은 리스크가 크며, 상황에 따라 불완전 복구(Incomplete Recovery)가 필요할 수 있음을 실습을 통해 체감.
