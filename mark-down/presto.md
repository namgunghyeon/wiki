# Presto정리
## 1.Presto란
Presto는 SQL on Hadoo 계열의 오픈소스 솔루션으로 Facebook에서 개발했으며, 현재 넷플릭스 등에서 사용되고 있는 오픈소스 분산된 서버에서 실행되며 SQL 처리 중 발생하는 중간 데이터 데이터는 메모리 기반으로 처리함으로써 기존 Hive On Mr보다 아주 빠른 성능으로 SQL을 처리할 수 있다. 또한 다양한 스토리지에서 저장된 데이터를 SQL을 이용하여 JOIN할 수 있는 장점이 있으며, 기가바이트부터 페타바이트까지 데이터를 다룰 수 있고, ANSI SQL를 지원한다.

중간 데이터는 모두 메모리에서만 처리하도록 되어 있기 때문에 배치 작업등에서는 제약이 있다.
정해진 시간까지만 실행하고 결과를 반환하거나, 정해진 에러율 내의 정확도로만 데이터를 반환하는 모드등을 지원하고 있다.

![presto](https://github.com/namgunghyeon/wiki/blob/master/images/presto/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-25%20%EC%98%A4%EC%A0%84%2012.50.39.png?raw=true)

인터페이스 역활을 하는 코디네이터와 실제 작업을 처리하는 Worker로 구성되어 있으며, 플러그인을 통해서 여러(hadoop, cassandra, kafak,  mysql)저장소를 사용할
있다.


![presto](https://github.com/namgunghyeon/wiki/blob/master/images/presto/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-29%20%EC%98%A4%EC%A0%84%2012.42.31.png?raw=true)

**클라이언트**
Presto Cli로 쿼리를 코디네이터에게 전달한다.

**코디네이터**
마스터 데몬으로 클라이언트로 부터 받은 쿼리의 분석 실행 계획을 세우고 가장 근접한 워커 노드에게 쿼리를 수행 시키고 모니터링하며 파이프라인을 통해서 실행된다.

**커넥터**
여러 저장소(Hive, HBase, Mysql, Cassandra ...)에 저장된 데이터를 가져올 수 있도록 연결해주며 사용자가 추가 구현할 수 있다. 커넥터를 사용하기 위해서는 해당 저장소의 메타 데이터가 제공되어야 한다.

**워커**
사용자가 입력한 쿼리를 수행하며 사용자에게 결과를 전달한다.

# 2.설치
http://getindata.com/blog/tutorials/tutorial-using-presto-to-combine-data-from-hive-and-mysql-in-one-sql-like-query/
https://prestodb.io/docs/current/installation/deployment.html
https://www.tutorialspoint.com/apache_presto/apache_presto_installation.htm
내용 참고

Cassandra, Mysql, kafka, hadoop, File

Presot 다운
```
용량이 크다....
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.156/presto-server-0.156.tar.gz
```

설치
```
tar -xvf presto-server-0.157.tar.gz

cd presto-server-0.157

해당 구조로 디렉토리와 파일을 생성해야 한다.
catalog 디렉토리 아래에 Connector정보들을 설정한다.
etc/
├── catalog
│   ├── cassandra.properties
│   ├── hive.properties
│   ├── kafka.properties
│   ├── mysql.properties
│   ├── postgresql.properties
│   └── redshift.properties
├── config.properties
├── jvm.config
├── log.properties
└── node.properties

```

config.properties <- 코디네이터 NODE 설정
```
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8080
query.max-memory=2GB
query.max-memory-per-node=1GB
discovery-server.enabled=true
discovery.uri=http://192.168.56.201:808
```

config.properties <- Worker  NODE 설정
```
coordinator=false
http-server.http.port=8080
query.max-memory=1GB
query.max-memory-per-node=1GB
discovery.uri=http://192.168.56.201:8080
```

jvm.config
```
-server
-Xmx16G
-XX:+UseG1GC
-XX:+UseGCOverheadLimit
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:OnOutOfMemoryError=kill -9 %p
```

cat log.properties
```
com.facebook.presto = INFO
```
node.properties
```
node.environment = production
node.id = 315404c0-a6ae-11e6-9137-0800276a26ef
node.data-dir = /home/ubuntu/node-data
```

catalog/mysql.properties
```
connector.name=mysql
connection-url=jdbc:mysql://192.168.56.115:3306
connection-user=root
connection-password=
```

실행
```
~/presto-server-0.157

bin/launcher start <- 백그라운드 실행
bin/launcher run <- 포어그라운드 실행
```

위 처럼 실행을 했지만
```
2016-11-10T04:02:40.451+0900	ERROR	Discovery-0	io.airlift.discovery.client.CachingServiceSelector	Cannot connect to discovery server for refresh (presto/general): Lookup of presto failed for http://192.168.56.201:8080/v1/service/presto/general
2016-11-10T04:02:40.786+0900	ERROR	Discovery-0	io.airlift.discovery.client.CachingServiceSelector	Cannot connect to discovery server for refresh (collector/general): Lookup of collector failed for http://192.168.56.201:8080/v1/service/collector/general
```
에러가 발생하면서
```
2016-11-10T04:02:45.853+0900	INFO	main	com.facebook.presto.server.PrestoServer	======== SERVER STARTED ========
2016-11-10T04:02:45.854+0900	ERROR	Announcer-0	io.airlift.discovery.client.Announcer	Cannot connect to discovery server for announce: Announcement failed with status code 404:
2016-11-10T04:02:45.855+0900	ERROR	Announcer-0	io.airlift.discovery.client.Announcer	Service announcement failed after 131.53ms. Next request will happen within 0.00s
2016-11-10T04:02:45.873+0900	ERROR	Announcer-1	io.airlift.discovery.client.Announcer	Service announcement failed after 8.11ms. Next request will happen within 1.00ms
2016-11-10T04:02:45.879+0900	ERROR	Announcer-2	io.airlift.discovery.client.Announcer	Service announcement failed after 3.28ms. Next request will happen within 2.00ms
2016-11-10T04:02:45.888+0900	ERROR	Announcer-3	io.airlift.discovery.client.Announcer	Service announcement failed after 3.61ms. Next request will happen within 4.00ms
2016-11-10T04:02:45.901+0900	ERROR	Announcer-2	io.airlift.discovery.client.Announcer	Service announcement failed after 4.69ms. Next request will happen within 8.00ms
2016-11-10T04:02:45.925+0900	ERROR	Announcer-1	io.airlift.discovery.client.Announcer	Service announcement failed after 6.15ms. Next request will happen within 16.00ms
2016-11-10T04:02:45.961+0900	ERROR	Announcer-0	io.airlift.discovery.client.Announcer	Service announcement failed after 3.63ms. Next request will happen within 32.00ms
```
정상적으로 동작하지 않는다.....
좀 더 찾아 봐야할 것 같다.

출처:
https://www.tutorialspoint.com/apache_presto/apache_presto_architecture.htm
