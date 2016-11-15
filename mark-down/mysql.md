# Mysql
## 1.Mysql이란
빠르고 견고한 관계형 데이터 베이스 관리 시스템(RDBMS)

## 2.Mysql구조
![mysql](https://raw.githubusercontent.com/namgunghyeon/wiki/a932214170e18fec4d9b7779bbfb8d52a391d9f9/images/mysql/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-15%20%EC%98%A4%ED%9B%84%2010.03.30.png)

##### Connection Pool
```
DB 이용시 연결을 위한 connection을 미리 만들어서 pool에서 저장해 연결 요청시 pool에 있는 connection을 리턴
- 실제로는 Thread Pool
```

##### SQL Support
```
- SELET, DML, DDL, 스토어드 프로시져, 트리거, 뷰 등을 지원
- SQL에 대한 함수 지원
```

##### Query Cache
```
최근 실행된 SQL 쿼리문의 결과를 저장해 이후 같은  SQL요청시 SQL을 쿼리 하지 않고 저장된 결과를 리턴
```
![mysql](https://raw.githubusercontent.com/namgunghyeon/wiki/a932214170e18fec4d9b7779bbfb8d52a391d9f9/images/mysql/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-15%20%EC%98%A4%ED%9B%84%2010.27.27.png)
##### Storage Engine
```
스토리지 엔진은 DBMS가 CRUD 작업시 기본적으로 사용되는 SW 컴포넌트, 수행결과에 대해서 디스크 스토리지 대상으로 저장 또는 읽어 오는 역할을 담당
```

#### mysql 쿼리 실행 구조
![mysql](https://raw.githubusercontent.com/namgunghyeon/wiki/a932214170e18fec4d9b7779bbfb8d52a391d9f9/images/mysql/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-15%20%EC%98%A4%ED%9B%84%2010.08.16.png)


```
1. 클라이언트는 서버로 SQL문을 전송한다.
2. SQL문은 받은 서버는 캐시를 확인하고 캐시가 되었다면 저장결과를 반환한다. 그렇지 않다면 다음 단계로 이동
3. 서버는 구문 분석 및 전처리기에서 쿼리에 구조적 문제가 없는지 화긴하고 옵티마이저는 SQL쿼리 최적화한다.
4. 쿼리 실행 엔진은 스토리지 엔진 APi를 호출하여 계획을 실행한다.
5. 서버는 클라이언트에게 결과를 반환한다.
```

##### Parser
```
사용자 요청으로 들어온 쿼리 문장을 토큰으로 분리해 트리 형태의 구조로 만든다. 기본 문법 오류는 이 과정에서 발견되어 사용자게게 오류 메시지를 전달하게 된다.
```
##### Preprocessor
```
각 토큰을 테이블 이름이나 컬럼 이름 또는 내장 함수와 매핑하여 해당 객체의 존재 여부와 객체의 접근 권한등을 확인하는 과정을 수행한다.
```

##### Optimizer
```
옵티마이져는 가장 적은 비용으로 가장 빠르게 처리할 수 있는 방법을 결정하기 위해 다양한 통계 자료를 활용한다.
- 사용자 요청의 SQL 수행을 위해, 후부군이 될만한 실행계획을 찾는다.
- Data Dictionary에 미리 수집해 놓은 오브젝트 통계 및 시스템 통계정보를 이요해 각 실행계획의 예상 비용을 산정한다.
  - 레코드 건수, 인텍스의 유니크 정보들
- 각 실행계획을 비교해서 최저비용을 갖는 하나를 선택한다.
```
##### Query Execution Engine
```
옵티마이저가 최적의 실행 계획을 결정하면 이를 직접 실행하는 부분을 담당한다. 스토리지 엔진의 API를 호출한다.

```


## 3.설치
```
sudo apt-get update
sudo apt-get install mysql-server
```

접속
```
mysql -u root -p
```

## 4.Replication

![mysql](https://raw.githubusercontent.com/namgunghyeon/wiki/70161422c189b5ed4d14fcf9a2396ba62f427bed/images/mysql/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-15%20%EC%98%A4%ED%9B%84%2010.42.50.png)

##### Master
DML 또는 DDL과 같은 구조 및 내용 변경 등을 바이너리 로그에 기록
```
DML(Data Manipulation Language)
INSERT, UPDATE, SELECT, DELETE ...
```
```
DDL(Data Definition Language)
CREATE , ALTER , DROP, RENAME ...
```

![mysql](https://raw.githubusercontent.com/namgunghyeon/wiki/70161422c189b5ed4d14fcf9a2396ba62f427bed/images/mysql/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-15%20%EC%98%A4%ED%9B%84%2010.42.25.png)

##### Slave(read-only)
슬레이브 서버의 I/O 스레드가 마스터 서버 접속을 통해 바이너리 로그 변경 내영을 받아와 릴레이 로그에 기록하면, 슬레이브 서버의 SQL 스레드가 릴레이 로그에 기록된 변경 내역을 바탕으로 재실행해 마스터와 동일한 상태 유지


## 5.Replication 설정
http://gywn.net/2012/02/mysql-replication-2/


### Mysql tuner
https://simplyopensource.blogspot.kr/2016/03/run-mysqltuner-on-aws-rds.html

출처:
http://imdsoho.tistory.com/entry/%EC%9D%B8%EB%8D%B1%EC%8A%A4-%EB%B0%8F-%EC%98%B5%ED%8B%B0%EB%A7%88%EC%9D%B4%EC%A0%80
http://sqlmvp.tistory.com/155
http://javapia.blogspot.kr/2010/07/mysql-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90.html
https://ahnndroid.wordpress.com/2015/04/29/mysql-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-1-%EC%A0%84%EC%B2%B4-%EA%B5%AC%EC%A1%B0-overall-arch/
http://egloos.zum.com/xxwony/v/25958
http://gywn.net/2011/12/mysql-replication-1/
