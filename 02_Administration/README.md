# 02. Administration

Oracle 인스턴스의 기동, 종료, 프로세스 상태 확인을 정리한 운영 기본 문서입니다.

## 1. Oracle 프로세스 확인

OS 레벨에서 PMON 프로세스를 확인하면 인스턴스가 기동 중인지 빠르게 판단할 수 있습니다.

```bash
ps -ef | grep pmon
```

주요 background process:

- PMON: 비정상 종료된 process 정리
- DBWn: buffer cache의 dirty block을 data file에 기록
- LGWR: redo log buffer를 redo log file에 기록
- CKPT: checkpoint 발생 시 control file과 data file header 정보 동기화

## 2. 관리자 접속

```bash
sqlplus / as sysdba
```

운영 작업은 SYSDBA 권한이 필요한 경우가 많으므로, 현재 접속 계정과 권한을 먼저 확인해야 합니다.

## 3. Startup 단계

Oracle 인스턴스는 다음 단계로 기동됩니다.

```text
NOMOUNT
-> parameter file 로드
-> SGA 할당
-> background process 기동

MOUNT
-> control file 로드
-> data file과 redo log 위치 확인

OPEN
-> 파일 정합성 확인
-> 사용자 접속 허용
```

실행:

```sql
STARTUP;
```

## 4. Shutdown 방식

권장 종료:

```sql
SHUTDOWN IMMEDIATE;
```

특징:

- 신규 접속 차단
- 진행 중 트랜잭션 rollback
- 안전한 종료

강제 종료:

```sql
SHUTDOWN ABORT;
```

주의:

- 즉시 프로세스를 종료합니다.
- 다음 startup 시 instance recovery가 필요할 수 있습니다.

## 학습 포인트

- startup 단계별로 읽는 파일이 다르기 때문에 장애 지점을 구분할 수 있습니다.
- shutdown 방식은 다음 기동 시 복구 비용에 영향을 줍니다.
- OS 프로세스와 DB 내부 상태를 함께 확인하는 습관이 중요합니다.
