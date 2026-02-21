## 🚀 ARCHIVELOG 모드 장애 대응 핵심 시나리오

서비스 가용성을 극대화하는 **온라인 복구**와 백업본이 없는 최악의 상황에서의 **데이터 재건** 역량을 실습한 기록입니다.

---

#### 🚨 Case 1. 운영 중 데이터 파일 유실 시 온라인 복구 (Zero-Downtime 기반)
> **포인트:** 데이터베이스 전체를 중단하지 않고, 장애가 발생한 특정 파일만 격리하여 복구함으로써 서비스 가용성을 보장합니다.

**1) 장애 상황 및 진단**
* `users01.dbf` 파일 삭제 후 새로운 테이블 생성 시 `ORA-01116`, `ORA-27041` 에러 발생.
* `v$recover_file` 조회 결과, 7번 파일(Users)이 `FILE NOT FOUND` 상태임을 확인.
* DB가 `MOUNT` 상태인 경우, 해당 데이터파일만 `OFFLINE` 처리하여 `OPEN` 상태로 진입 가능함을 실습.

**2) 복구 절차 (Online Recovery)**
```sql
-- 1. 장애가 발생한 데이터파일을 격리 (Offline 처리)
ALTER DATABASE DATAFILE 7 OFFLINE;

-- 2. 데이터베이스 오픈 (다른 서비스는 정상화)
ALTER DATABASE OPEN;

-- 3. 최신 백업본에서 유실된 파일 복원 (OS 레벨)
-- [oracle@oel7v9r1 ~]$ cp /home/oracle/backup/open_bkp/users01.dbf /u01/app/oracle/oradata/ORCL/

-- 4. 아카이브 로그 및 리두 로그를 적용하여 장애 시점까지 완전 복구 수행
-- 실습 로그 상 Sequence #13부터 #18까지의 아카이브 로그가 순차적으로 적용됨
RECOVER TABLESPACE USERS;
-- Specify log: {<RET>=suggested | filename | AUTO | CANCEL} -> 'AUTO' 입력

-- 5. 복구 완료 후 테이블스페이스를 다시 온라인으로 전환
ALTER TABLESPACE USERS ONLINE;
```
**3) 검증 및 결과**

- `v$datafile`에서 모든 파일의 STATUS가 ONLINE 및 SYSTEM으로 복구되었음을 확인.

- 장애 발생 전 입력했던 데이터(hr.emp7 등)가 유실 없이 조회됨으로써 완전 복구(Complete Recovery) 성공을 확인.

**핵심 학습 포인트 (Lessons Learned)**
- `비정상 종료 대응`: 리두 로그와 아카이브 로그가 모두 존재할 경우, MOUNT 단계에서 문제가 된 파일만 OFFLINE으로 빼고 나머지를 먼저 OPEN하여 서비스 다운타임을 최소화할 수 있습니다.

- `복구 자동화`: RECOVER 명령어 실행 시 AUTO 옵션을 활용하여 여러 개의 아카이브 로그 파일을 수동 입력 없이 자동으로 적용하는 효율적인 방법을 습득했습니다.

