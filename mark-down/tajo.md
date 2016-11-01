# Tajo정리
## 1.Tajo란
하둡 기반의 대용량 데이터 웨어 하우스 시스템 SQL 표준, 전체 질의 분산 처리
TajoMaster와 TajoWorker로 구성되며, 마스터에 문제가 생길 시 정상적으로 사용할 수 있도록 슬레이브 서버에 QueryMaster를 두어 정상적으로 처리가
가능하도록 구성되어 있다.
![tajo](https://github.com/namgunghyeon/wiki/blob/master/images/tajo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-14%20%EC%98%A4%EC%A0%84%2012.33.44.png?raw=true)

**클라이언트**
- tsql
- JDBC드라이버

**타조 마스터**
- 타조 클러스터의 마스터 역활
- 클라이언트 API를 RPC로 제공
- 질의를 파싱해서 논리적인 질의 실행 계획을 수집하는 플래너를 제공
- 수립된 질의 최적화
- 쿼리 마스터를 관리. 질의 실행 요청이 들어올 경우, 유효한 타조워커 중에 하나를 쿼리 마스터로 선정
- 전체 클러스터의 자원을 관리


**쿼리 마스터**
- 타조 마스터 생성한 논리 실행 계획을 분산 실행 계획으로 변환, 변환된 분산 실행 계획을 제어하고, 태스크를 스케쥴링한다.
- 질의 하나당 쿼리 마스터 하나가 생성된다.

**타조 워커**
- 타조워커는 타조워커와 쿼리마스터 두 가지 역활을 수행한다.


## 2.질의 실행 계획 과정
사용자가 작성한 질의 대한 구분 분석을 요청, 질의 구문 분석기는 클라이언트가 전달한 질의 대수학 표현식으로 생성, JSON형태
![tajo](https://github.com/namgunghyeon/wiki/blob/master/images/tajo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-14%20%EC%98%A4%EC%A0%84%201.15.42.png?raw=true)


## 3.설치
http://blrunner.com/101 참고해서 설치할 예정
![tajo](https://github.com/namgunghyeon/wiki/blob/master/images/tajo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-25%20%EC%98%A4%EC%A0%84%2012.22.52.png?raw=true)
