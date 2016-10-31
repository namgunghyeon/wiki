# Flume 정리
## 1. Flume란
Flume은 스트림 지향 데이터 플로우를 기반으로해 지정된 모든 서버로부터 로그를 수집한 후 하둡 HDFS와 같은 중앙 저장소에 적재하는 시스템이다.


***시스템 신뢰성*** : 장애가 발생시 로그의 유실없이 전송할 수 있는 기능

***시스템 확장성*** : Agent의 추가 및 제저가 용의

***관리 용의성*** : 한결한 구조로 관리가 용이

***기능확장성*** :  새로운 기능을 쉽게 추가할 수 있음

클라우데라에서 개발하던 0.X 버전은 Flume OG라고 지칭하고 아파치 오픈소스로 이관된 이후의 1.X버전은 Flume NG라고 부른다. Flume OG에서 NG로 변경이 되면서 Agent, Collector, Master로 구분지어 지던 부분을 하나의 Agent로 통합해 단순해졌으며, 확장성과 높은 자유도를 가지게 되었다.

![Flume](https://github.com/namgunghyeon/wiki/blob/master/images/flume/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.02.28.png?raw=true)


## 2.구조

***Source*** : Event를 받아 입력된 정보를 Sink에 전달한다.

***Channel*** : Source와 Sink의 Dependency를 제거하고 장애에 대비하기 위하여 중간 채널의 제공하며 Source는 Channel에 Event정보를 저장하며 Sink는 채널로부터 받아 처리한다.

***Sink*** : 채널로부터 Source가 전달한 Event정보를 하둡 HDFS에 저장하거나 다음 Tire의 Agent또는 DB로 전달한다. 지정된 프로토콜의 Type에 따른 처리를 진행한다.

![Flume](https://github.com/namgunghyeon/wiki/blob/master/images/flume/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.02.38.png?raw=true)

Avro를 통해서 Agent간 연결해서 사용
Avro : 데이터 직려화를 하는 프레임워크(비슷한 종류로는 Protocol Buffer, Thrift, Avro는 하둡을 만든 더크 커팅이 개발) 속도는 빠르지 않지만 크기가 작다.
https://avro.apache.org/

![Flume](https://github.com/namgunghyeon/wiki/blob/master/images/flume/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.02.42.png?raw=true)


대용량 로그 처리를 위한 일반적인 구조

![Flume](https://github.com/namgunghyeon/wiki/blob/master/images/flume/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.02.47.png?raw=true)



***Source 종류***
```
spooling Directory Source : 디렉토리에 새롭게 추가되는 파일을 데이터로 사용

Exec Source : 명령행을 실행하고 콘솔 출력 내용을 데이터로 입력으로 사용

Avro Source : 외부 Avro 클라이언트에서 전송하는 데이터를 수신해서 입력으로 사용

JMS Source : JMS 메시지를 입력으로 사용

Syslog Source : 시스템 로그를 입력으로 사용

NetCat Source : TCP로 라인 단위 압력 받음
```

***Sink 종류***

```
HDFS Sink : HDFS에 데이터를 파일로 씀

HBase Sink : HBase에 데이터를 씀

Avro Sink: 다른 Avro 서버(Avro Source)에 이벤트로 전달함

Thrift Sink: 다른 Thrift(Thrift Source)에 이벤트로 전달함

File Roll Sink: 로컬 파일에 데이터를 씀

MorphlineSolr Sink : 데이터를 변환해서 Solr 서버에 씀

ElasticSearch Sink : 데이터를 변환해서 ElasticSearch클러스터에 씀

Null Sink : 이벤트를 버럼
```


## 3.설치(1.6.0)

***Apache Flume 다운***
http://apache.mirror.cdnetworks.com/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz
***Source(netcat) <- Channel(memory) -> Sink(logger) 테스트***

```
Flume 설정(Source, Channel, Sink)
/home/nkh/apache-flume-1.6.0-bin/conf
cp flume-conf.properties.template flume.conf

아래와 같이 설정
# Naming the components on the current agent
NetcatAgent.sources = Netcat
NetcatAgent.channels = MemChannel
NetcatAgent.sinks = LoggerSink

# Describing/Configuring the source
NetcatAgent.sources.Netcat.type = netcat
NetcatAgent.sources.Netcat.bind = localhost
NetcatAgent.sources.Netcat.port = 56565

# Describing/Configuring the sink
NetcatAgent.sinks.LoggerSink.type = logger

# Describing/Configuring the channel
NetcatAgent.channels.MemChannel.type = memory
NetcatAgent.channels.MemChannel.capacity = 1000
NetcatAgent.channels.MemChannel.transactionCapacity = 100

# Bind the source and sink to the channel
NetcatAgent.sources.Netcat.channels = MemChannel
NetcatAgent.sinks.LoggerSink.channel = MemChannel

Flume 실행
.bin/flume-ng agent -c conf -f conf/flume.conf -n NetcatAgent -Dflume.root.logger=INFO,console


```

#### 테스트
```
동일한 서버에서  curl telnet://localhost:56565 접속후 키보드로 입력하는 데이터 플럼으로  전달
```

![Flume](https://github.com/namgunghyeon/wiki/blob/master/images/flume/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.02.57.png?raw=true)

![Flume](https://github.com/namgunghyeon/wiki/blob/master/images/flume/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.03.02.png?raw=true)

***출처:
https://kimpaper.github.io/2015/10/29/flume-setting-sample/
http://www.dbguide.net/knowledge.db?cmd=specialist_view&boardUid=180258&boardConfigUid=108&boardStep=0&categoryUid=
http://www.slideshare.net/madvirus/flume-29149433
http://flume.apache.org/FlumeUserGuide.html#topology-design-considerations
http://www.slideshare.net/ssuser7e0f1e/avro-31773255***

***설치 및 설정 관련:
http://devzeroty.tistory.com/entry/Flume-%EC%84%A4%EC%B9%98-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95-%EB%B0%8F-%ED%85%8C%EC%8A%A4%ED%8A%B8
http://www.gooper.com/ss/bigdata/270701
http://www.tutorialspoint.com/apache_flume/apache_flume_netcat_source.htm***
