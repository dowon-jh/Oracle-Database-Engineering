# 01. Environment Setup

Oracle 19c 인스턴스를 운영하기 위한 Linux 환경 변수 설정과 기본 접속 흐름을 정리한 문서입니다.

## 목적

Oracle 엔진과 인스턴스에 접근하기 위해 필요한 환경 변수를 `.bash_profile`에 설정하고, 터미널 접속 시 자동으로 Oracle 실행 환경이 로드되도록 구성합니다.

## 주요 환경 변수

| 변수 | 역할 |
| --- | --- |
| `ORACLE_BASE` | Oracle 소프트웨어 설치 기준 경로 |
| `ORACLE_HOME` | Oracle 엔진이 설치된 경로 |
| `ORACLE_SID` | 접속할 Oracle 인스턴스 식별자 |
| `PATH` | SQL*Plus 등 Oracle 실행 파일 경로 포함 |
| `NLS_LANG` | 문자셋과 언어 환경 설정 |

## 설정 예시

```bash
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/dbhome_1
export ORACLE_SID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
```

## 적용 및 확인

```bash
source ~/.bash_profile
echo $ORACLE_SID
sqlplus / as sysdba
```

## 학습 포인트

- `ORACLE_SID`가 잘못되면 원하는 인스턴스가 아닌 다른 인스턴스에 접속하거나 접속 자체가 실패할 수 있습니다.
- `ORACLE_HOME`은 SQL*Plus, DBCA, listener 등 Oracle 관리 도구 실행 경로와 연결됩니다.
- 운영 환경에서는 계정별 profile 설정이 DB 관리 작업의 출발점입니다.
