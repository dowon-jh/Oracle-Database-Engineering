# ARCHIVELOG Recovery Scenarios

ARCHIVELOG 모드에서 데이터 파일 유실이 발생했을 때 온라인 복구로 서비스 다운타임을 줄이는 절차를 정리한 문서입니다.

## Case 1. 운영 중 데이터 파일 유실

상황:

- `users01.dbf` 파일 삭제
- 신규 테이블 생성 또는 접근 시 `ORA-01116`, `ORA-27041` 발생
- `v$recover_file`에서 파일 유실 확인

진단:

```sql
SELECT * FROM v$recover_file;
```

## 온라인 복구 흐름

```text
손상 data file 확인
-> 해당 data file offline
-> database open
-> 백업본 복원
-> archive log와 redo log 적용
-> tablespace online
```

복구 절차:

```sql
ALTER DATABASE DATAFILE 7 OFFLINE;
ALTER DATABASE OPEN;
RECOVER TABLESPACE USERS;
ALTER TABLESPACE USERS ONLINE;
```

OS 레벨에서는 백업본을 원래 data file 위치로 복사합니다.

```bash
cp /home/oracle/backup/open_bkp/users01.dbf /u01/app/oracle/oradata/ORCL/
```

## 검증

```sql
SELECT file#, status, name
FROM v$datafile;
```

복구 전 입력했던 데이터가 조회되면 완전 복구가 성공한 것입니다.

## 학습 포인트

- ARCHIVELOG 모드는 archive log를 통해 장애 시점까지 변경분을 재적용할 수 있습니다.
- 특정 tablespace만 offline 처리해 나머지 서비스는 open 상태로 유지할 수 있습니다.
- `RECOVER ... AUTO` 방식을 사용하면 여러 archive log를 순차 적용할 수 있습니다.
- 운영 환경에서는 백업 주기, archive log 보존 정책, 복구 목표 시점이 함께 설계되어야 합니다.
