# 03. SQL and Monitoring

Oracle Dynamic Performance Views를 활용해 인스턴스, 세션, redo log 상태를 진단하는 문서입니다.

## 1. 인스턴스 상태 확인

```sql
SELECT instance_name, host_name, status, startup_time
FROM v$instance;
```

상태 의미:

- `OPEN`: 사용자 접속 가능 상태
- `MOUNTED`: control file 로드 완료, 복구 작업 가능
- `STARTED`: parameter file 로드와 SGA 할당 완료

주요 파라미터 확인:

```sql
SHOW PARAMETER sga;
SHOW PARAMETER control;
```

## 2. 세션 모니터링

현재 접속 중인 사용자와 애플리케이션 연결 상태를 확인합니다.

```sql
SELECT sid, serial#, username, status, program, machine
FROM v$session
WHERE status = 'ACTIVE'
  AND username IS NOT NULL;
```

컬럼 활용:

- `sid`, `serial#`: 세션 kill 시 식별자
- `program`: 접속 프로그램 또는 애플리케이션
- `machine`: 접속한 서버 또는 클라이언트

백엔드 관점에서는 connection pool 고갈, 장시간 active session, 비정상 세션 누적을 추적할 때 유용합니다.

## 3. Redo Log 상태 확인

```sql
SELECT group#, sequence#, bytes / 1024 / 1024 AS mb, members, status
FROM v$log;
```

상태 의미:

- `CURRENT`: LGWR이 현재 기록 중인 log group
- `ACTIVE`: checkpoint가 끝나지 않아 복구에 아직 필요한 log group
- `INACTIVE`: 재사용 가능한 log group

## 학습 포인트

- Oracle 모니터링은 상태 조회 쿼리의 의미를 해석하는 것이 중요합니다.
- `v$session`은 애플리케이션 서버와 DB 사이의 연결 문제를 분석하는 핵심 뷰입니다.
- redo log가 계속 `ACTIVE`에 머물면 checkpoint 지연이나 I/O 병목을 의심할 수 있습니다.
