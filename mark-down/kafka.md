# Kafka 정리
## 1.Kafka란
Apache Kafka는 Linkedin에서 개발된 분산 메시징 시스템 대용량의 실시간 로그처리에 특화된 아키텍쳐 설계를 통해 기존 메시징 시스템보다 우수한 TPS를 보여준다. **TPS(Transaction Per Second)**: 초당 처리할 수 있는 트랜잭션의 양
![Kafka](https://github.com/namgunghyeon/wiki/blob/master/images/kafak/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.03.47.png?raw=true)

## 2.구조
Kafaka는 발행 - 구독 모델을 기반으로 동작하며 Producer, Consumer, Broker로 구성된다.

![Kafka](https://github.com/namgunghyeon/wiki/blob/master/images/kafak/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.03.50.png?raw=true)

Kafka의 Broker는 Topic을 기준으로 메시지를 관리한다.
1. Producer는 특정 Topic의 메시지를 생성한 뒤 해당 메시지를 Broker에게 전달한다.
2. Broker가 전달 받은 메시지를 Topic별로 분류
3. 해당 Topic을 구독하는 Consumer들이 메시지를 가져가 처리하게 된다.

Kafka는 확장성, 고가용성을 위하여 Broker들이 클러스터로 구성되어 동작한다. Broker가 1개 밖에 없을 때도 클러스터로 동작한다.

**Broker**: 카프카의 서버, 1대 이상의 서버들로 클러스터링 가능
**Topic**: 유사한 메시지들의 집합. 프로듀서는 메시지를 전달할 토픽을 반드시 지정해야한다.
**Partiion**: 로드밸런싱을 목적으로 토픽을 논리적으로 분할
**Replication**: Fault Tolerance 위해 파티션 단위로 복제한다.
**Producer**: 메시지 제공자
**Consumer**: 메시지 소비자

### Topic
Topic은 Partiion이라는 단위로 쪼개져 클러스터의 각 서버들에 분산되어 저장되고, 고가용성을 위하여 복제 설정을 할 경우 각 서버에 Partition단위로 분산되어 저장되어 복제되고 장애가 발생되면 Partition단위로 Fail Over가 수행된다.

![Kafka](https://github.com/namgunghyeon/wiki/blob/master/images/kafak/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.03.55.png?raw=true)

각 Partition은 0부터 1씩 증가하는 Offset값을 메시지에 부여하는데 이 값은 각 Partition내에서 메시지를 식별하는 ID로 사용된다. Offset값은 Partition마다 별도로 관리 되므로 Topic내에서 메시지를 식별할 때는 Partition 번호와 Offset 값을 함께 사용한다.

### Partition
Kafka에서 고가용성을 위해 각 Partition을 복제하여 클러스터에 분산시킬 수 있다.
아래 그림은 해당 Topic의 Replication Factor를 3으로 설정한 상태의 N클러스터이다. 각 Partition들은 3개의 Replica를 가지며 각 Replica는 R0, R1, R2로 표시되어 있다.

![Kafka](https://github.com/namgunghyeon/wiki/blob/master/images/kafak/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.04.00.png?raw=true)

Replication Factor를 N으로 설정할 경우 N개의 replica는 1개의 Leader와 N-1(Leader를 뺀 나머지)개의 Follower로 구성된다. 각 Partition에 대한 읽기 와 쓰기는 모두 Leader에서 이루어지고, Follower는 Leader를 복제하기만한다. Leader에서 장애가 발생할 경우 Follower중 하나가 새로운 Leader가 된다. F + 1개의 replica를 가진 Topic이 F개의 장개 까지 버틸 수 있도록 한다.(Leader하나만 남을 때 까지 버틴다.)

### Consumer와 Consumer Group
메시징 모델은 큐 모델과 구독 모델로 나뉜다. 큐 모델은 메시지가 쌓여 있는 큐로부터 메시지를 가져와 Consumer Pool에 있는 Consumer 중 하나에 메시지를 할당하는 방식이고, **발생 - 구독 모델은 Topic을 구독하는 모든 Consumer에게 메시지를 브로드캐스팅하는 방식이다.**

Kafaka의 Partition은 Consumer Group당 하나의 Consumer의 접근만을 허용하며, 해당 Consumer를 Partition Owner라고 부른다. 동일한 Consumer Group에 속하는 Consumer끼리는 동일한 Partition에 접근할 수 없다.

Consumer가 추가/제거되면 추가/제거된 Consumer가 속한 Consumer Group내의 Consumer 들의 Partition Rebalancing이 발생하고 Broker가 추가/제거되면 전체 Consumer Group에서 Partition이 재분배가 발생한다.

Consumer Group을 구성하는 Consumer의 수가 Partition의 수보다 적으면 하나의 Consumer가 여러개의 Partition을 소유하게 되고, 반대로 Consumer의 수가 Partition 수보다 많으면 Consumer는 메시지를 처리하지 않게 되므로 Partition 개수와  Consumer 수의 적절한 설정이 필요하다.

![kafka](https://github.com/namgunghyeon/wiki/blob/master/images/kafak/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.04.05.png?raw=true)

**위 그림과 같이 Consumer Group에 다수의 Consumer를 할당하면 각 Consumer마다 별도의 Partition으로 부터 메시지를 받아 오기 때문에, Consumer Group은 Consumer Group은 큐 모델로 동작한다.**

**단일 Consumer로 이루어진 Consumer Group을 활용하면 다수의 Consumer가 동일한 Partition에 동시에 접근하여 동일한 메시지를 엑세스하기 때문에 발행 - 구독 모델을 구성할 수 있다.**

하나의 Consumer에 의하여 독점적으로 Partition이 액세스 되기 때문에 동일 Parition 내에 미시지는 Partition에 저장된 순서대로 처리된다. 다른 Partition에 속한 메시지의 순차적 처리는 보장되지 않기 때문에, 특정 Topic의 전체 미시지가 발생 시간 순으로 처리되어야 할 경우 해당 Topic이 하나의 Partition만 가지도록 설정해야 한다.

이상적인 구조는 하나의 Partition을 하나의 Consumer가 담당하는것이다. Consumer 개수가 Partition 개수보다 적으면 Consumer가 여러 Partition을 담당하게 되어 메시지 순서를 보장할 수 없다. 메시지 처리 순서가 중요한 경우 하나의 Partition에 하나의 Consumer로 구성되록 신경 써야 한다.

액세스된 데이터는 바로 삭제가되지 않는 구조로 일정기간 동안 다시 읽을 수 있다.


### 파일 시스템을 사용한 고성능 디자인
Kafka는 기존 메시징 시스템과 달리 메시지를 메모리대신 파일 시스템에 쌓아두고 관리한다. 하드디스크의 순차적읽기는 메모리의 램덤 읽기 성느보다 7배 정도 느리다.(하드 디스크의 랜덤 읽기 성능은 메모리의 랜덤 읽기 보다 10만배 느리다.)

Kafka는 메모리에 별도의 캐시를 구현하지 않고 OS의 페이지 캐시에 모두 위임한다. OS가 알아서 서버의 유휴 메모리를 페이지 캐시로 활용하여 앞으로 필요한 것으로 예상되는 메시지들을 읽어 들여 디스크 읽기를 향상 시킨다.

Kafka의 메시지는 디스크로부터 순차적으로 읽혀지기 때문에 하드디스크이의 랜덤 읽기 성능에 대한 단점을 보완하고 동시에 OS에 페이지 캐시를 효과적으로 사용할 수 있다.

##### Zero-Copy
Kafka에서는 파일 시스템에 저장된 메시지를 네트워크를 통해 Consumer에게 전송할 때 Zero-Copy 기법을 사용한다.
커널 모드와 유저 모드간의 데이터 복사가 이뤄지는데 Zero-Copy를 통해서 커널 모드와 유저 모드간의 데이터 통신을 업애고 커널 모드에 데이터를 전송한다.

![kafka](https://github.com/namgunghyeon/wiki/blob/master/images/kafak/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.04.10.png?raw=true)

## 3.설치
http://apache.mirror.cdnetworks.com/kafka/0.10.0.1/kafka_2.11-0.10.0.1.tgz
**Zookeeper클러스터 구축**
```
카프카에는 주키퍼 패키기지가 기본적으로 들어있다.
/home/nkh/kafka_2.11-0.10.0.1/config/zookeeper.properties

dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0

initLimit=5
syncLimit=2
server.1=10.128.0.7:2888:3888 (104.155.190.249)
server.2=10.128.0.8:2888:3888 (104.198.30.2)
server.3=10.128.0.9:2888:3888(104.198.21.36)

server.#에서 #은 인스턴스 ID로 각 서버마다 1,2,3으로 설정을 해준다.
dataDir 디렉토리 아래 myid라는 파일에 명시 해주어야 한다.
각 서버마다 bin/zookeeper-server-start.sh config/zookeeper.properties 으로 실행 시킨다.

```

**Kafka클러스터 구축**
```
/home/nkh/kafka_2.11-0.10.0.1/config/server.properties
맨위 설정
broker.id=# 에서 #은 주키퍼에 설정했던 myid값과 동일하게 설정(모든 서버 설정)
위에서 설정했던 주키퍼 주소
zookeeper.connect=10.128.0.7:2181,10.128.0.8:2181,10.128.0.9:2181
104.155.190.249:2181,104.198.30.2:2181,104.198.21.36:2181

실행
bin/kafka-server-start.sh config/server.properties

Topic 생성하기
3대의 서버중 한대에서 설정하면 모든서버에게 설정이 된다.
bin/kafka-topics.sh --create --zookeeper 10.128.0.9:2181 --replication-factor 3 --partitions 20 --topic test

Topic 리스트 확인
bin/kafka-topics.sh --list --zookeeper 10.128.0.9:2181

Topic의 replicat와 partition정보 확인
bin/kafka-topics.sh --describe --zookeeper 10.128.0.9:2181 --topic test

Topic 설정 변경하기
bin/kafka-topics.sh --alter --zookeeper 10.128.0.9:2181 --topic test --partitions 40
```

**Topic 자동 생성시 주의할 점**
```
존재 하지 않는 Topic에 대해서 생산하거나 소비할 경우 Broker의 설정값 따라 Topic이 자동으로 생성된다.
1개의 Partition과 replica를 가지는 Topic이 생성되어 이 값은 운영에 적합하지  않아 Topic을 사용하기 전에 kafka-topics.sh으로 미리 Topic을 생성하는게 좋다.
```

**테스트**
```
생산자 테스트
bin/kafka-console-producer.sh --broker-list 10.128.0.9:9092 --topic test
소비자 테스트
bin/kafka-console-consumer.sh --zookeeper 10.128.0.9:2181 --topic test --from-beginning
```

```
주키퍼 설정 가이드
http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_configuration
카프카 가이드
http://kafka.apache.org/documentation.html
```

**출처:
http://m.dbguide.net/about.db?cmd=view&boardConfigUid=19&boardUid=183426
http://epicdevs.com/17
http://www.joinc.co.kr/w/man/12/zookeeper/Watcher**
