# .bash_profile

# Oracle software가 설치된 최상위 경로
export ORACLE_BASE=/u01/app/oracle

# Oracle 엔진이 설치된 경로
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/dbhome_1

# 접속할 데이터베이스의 고유 식별자 (SID)
export ORACLE_SID=orcl

# 오라클 실행 파일 경로를 시스템 PATH에 추가
export PATH=$PATH:$ORACLE_HOME/bin

# 한글 깨짐 방지 및 언어 설정
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8

#사용자 홈 디렉토리의 .bash_profile 파일에 아래 내용을 추가하여, 터미널 접속 시 자동으로 오라클 환경이 로드되도록 구성
#수정한 내용을 즉시 적용하고, 변수가 정상적으로 등록되었는지 확인하는 과정입니다.
#설정 적용: source .bash_profile

#변수 확인: echo $ORACLE_SID

#접속 테스트: sqlplus / as sysdba
