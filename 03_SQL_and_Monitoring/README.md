##  3. DB 상태 진단 및 실시간 모니터링 (Dynamic Performance Views)

Oracle의 동적 성능 뷰(V$ Views)와 명령어를 활용하여 인스턴스의 현재 상태와 세션 흐름을 파악하는 방법입니다.


#### 1) 인스턴스 정보 확인 (V$INSTANCE & show parameter)
DB 서버가 정상적으로 작동 중인지, 어떤 환경으로 설정되어 있는지 확인합니다.

* **인스턴스 기동 상태 확인:**
    ```sql
    SELECT instance_name, host_name, status, startup_time FROM v$instance;
    ```
    > **Status 상세:**
    > * `OPEN`: 정상 서비스 상태
    > * `MOUNTED`: 컨트롤 파일 로드 완료 (복구 작업 등 가능)
    > * `STARTED`: 파라미터 파일 로드 완료

* **주요 파라미터 조회:**
    ```sql
    SQL> show parameter sga;     -- 메모리 구조(SGA) 크기 확인
    SQL> show parameter control; -- Control File 경로 및 개수 확인
    ```

---

####  2) 실시간 세션 및 연결 모니터링 (V$SESSION)
현재 데이터베이스에 접속한 유저와 프로세스의 상태를 추적합니다. **백엔드 개발 시 DB Connection Pool 이슈**가 발생했을 때 원인을 찾는 핵심 데이터입니다.

* **활성 세션 조회:**
    ```sql
    SELECT sid, serial#, username, status, program, machine 
    FROM v$session 
    WHERE status = 'ACTIVE' AND username IS NOT NULL;
    ```
    * `sid`, `serial#`: 특정 세션을 강제 종료(Kill)할 때 식별자로 사용합니다.
    * `program`, `machine`: 어떤 애플리케이션 서버에서 접속했는지 식별합니다.

---

####  3) 로그 스위치 및 리두 로그 상태 (V$LOG)
데이터 변경 이력을 기록하는 리두 로그의 순환 구조를 확인합니다.

* **로그 그룹 상태 조회:**
    ```sql
    SELECT group#, sequence#, bytes/1024/1024 as MB, members, status FROM v$log;
    ```
    * `CURRENT`: 현재 LGWR에 의해 로그가 기록되는 중인 그룹입니다.
    * `ACTIVE`: 체크포인트가 지연되어 아직 데이터 파일에 기록되지 않은 변경사항이 있음을 의미하며, 지속될 경우 성능 병목을 의심할 수 있습니다.
