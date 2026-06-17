# NOARCHIVELOG Recovery Scenarios

NOARCHIVELOG 모드에서 발생할 수 있는 대표 장애와 복구 절차를 정리한 문서입니다. 이 모드는 archive log가 없기 때문에 복구 가능 범위가 제한되며, redo log가 덮어쓰기 전인지 여부가 중요합니다.

## Case 1. 데이터 파일 손상

상황:

- `users01.dbf` 파일 삭제
- DB 기동 시 `ORA-01157` 발생
- `v$recover_file`에서 손상된 파일 확인

진단:

```sql
SELECT * FROM v$recover_file;
```

복구 흐름:

```sql
ALTER DATABASE DATAFILE 7 OFFLINE DROP;
ALTER DATABASE OPEN;
RECOVER DATAFILE 7;
ALTER DATABASE DATAFILE 7 ONLINE;
```

핵심 포인트:

- NOARCHIVELOG 모드라도 redo log가 남아 있으면 제한적으로 완전 복구가 가능할 수 있습니다.
- 백업 이후 변경분이 redo log에 남아 있지 않으면 데이터 손실이 발생할 수 있습니다.

## Case 14. Control File 유실

상황:

- `control01.ctl` 삭제
- startup 시 `ORA-00205` 발생

복구 흐름:

```sql
ALTER DATABASE BACKUP CONTROLFILE TO TRACE AS '/home/oracle/cre_con.sql';
```

trace 파일의 `CREATE CONTROLFILE` 스크립트를 수정해 NOMOUNT 단계에서 control file을 재생성합니다.

사후 조치:

```sql
RECOVER DATABASE;
ALTER DATABASE OPEN;
ALTER TABLESPACE TEMP ADD TEMPFILE '...' REUSE;
```

핵심 포인트:

- control file은 DB 구조 메타데이터를 담기 때문에 다중화가 중요합니다.
- trace 기반 복구는 백업 control file이 없을 때 사용할 수 있는 최후 수단입니다.

## Case 24. Current Redo Log File 삭제

상황:

- `CURRENT` 상태의 redo log group 삭제
- DB open 실패

복구 예시:

```sql
STARTUP MOUNT;
ALTER DATABASE CLEAR LOGFILE GROUP 3;
ALTER DATABASE OPEN;
```

핵심 포인트:

- current redo log 유실은 데이터 손실 위험이 큽니다.
- `CLEAR LOGFILE`은 장애 시점 이후 변경분 손실 가능성을 이해한 뒤 사용해야 합니다.

## 운영 관점 정리

- NOARCHIVELOG 모드는 복구 범위가 좁아 운영 DB에는 부적합한 경우가 많습니다.
- 장애 파일 종류가 data file, control file, redo log 중 무엇인지 먼저 구분해야 합니다.
- 복구 전 startup 단계, alert log, `v$recover_file`, redo log 상태를 함께 확인합니다.
