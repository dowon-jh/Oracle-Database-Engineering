## 2. Oracle 인스턴스 제어 및 프로세스 모니터링

리눅스 터미널 환경에서 오라클 데이터베이스의 상태를 확인하고, 인스턴스를 기동 및 종료하는 실무적인 운영 방법입니다.


#### 1) 오라클 주요 프로세스 확인
OS 레벨에서 오라클 백그라운드 프로세스가 정상적으로 구동 중인지 체크합니다.

```bash
# PMON 프로세스 확인 (인스턴스 기동 여부 파악)
ps -ef | grep pmon
```
💡 주요 백그라운드 프로세스 역할

- PMON (Process Monitor): 프로세스 감시 및 장애 시 정리 수행. (존재 시 인스턴스 기동 의미)

- DBWn (Database Writer): 버퍼 캐시의 데이터를 실제 데이터 파일에 기록.

- LGWR (Log Writer): 로그 버퍼의 내용을 리두 로그 파일에 기록.

- CKPT (Checkpoint): DB 상태 정보 동기화 및 체크포인트 신호 전달.



#### 2) 인스턴스 기동 및 종료 (Startup / Shutdown)
sqlplus 관리자 권한으로 접속하여 데이터베이스 상태를 제어합니다.
```
# 관리자 권한 접속
sqlplus / as sysdba
```
✅ 인스턴스 기동 단계 (Startup Flow)

- NOMOUNT: Parameter File 로드, SGA 메모리 할당 및 프로세스 기동

- MOUNT: Control File 로드, 데이터 파일 및 리두 로그 위치 파악

- OPEN: 파일 정합성 체크 후 사용자 접속 허용

✅ 인스턴스 종료 방식 (Shutdown)

- SHUTDOWN IMMEDIATE: (권장) 트랜잭션 롤백 및 안전한 종료.

- SHUTDOWN ABORT: 프로세스 강제 종료. (다음 기동 시 Instance Recovery 필요)



