## 1. Oracle 환경 변수 설정 (.bash_profile)

리눅스 환경에서 오라클 데이터베이스 인스턴스에 접속하고 관리하기 위해 필수적인 환경 변수를 설정한 기록입니다.

**주요 설정 내용:**
- `ORACLE_BASE`: Oracle software가 설치된 최상위 경로
- `ORACLE_HOME`: Oracle 엔진이 설치된 경로
- `ORACLE_SID`: 접속할 데이터베이스의 고유 식별자
- 설정 적용: source .bash_profile
- 변수 확인: echo $ORACLE_SID
- 접속 테스트: sqlplus / as sysdba

```bash
# .bash_profile 설정 내용
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/dbhome_1
export ORACLE_SID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8

- 사용자 홈 디렉토리의 .bash_profile 파일에 아래 내용을 추가하여, 터미널 접속 시 자동으로 오라클 환경이 로드되도록 구성
- 수정한 내용을 즉시 적용하고, 변수가 정상적으로 등록되었는지 확인하는 과정입니다.
```
