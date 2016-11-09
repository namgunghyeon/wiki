# Cassandra정리
## 1.Cassandra란
Cassandra는 Scalability와 High Avaliablility에 최적화된 대표적인 분산형 Data Storeage이다. Consistent hashing을 이용한 Ring 구조와 Gossip Protocol을 구현했으며, 각 노드 장비들의 추가, 제거 등이 자유롭고, 데이터 센터까지 고려 할 수 있는 데이터 복제 정책을 사용하여 안정성 축면에서 많은 장점이 있다.
하지만 Join이나 Transaction을 지원하지 않고, Index등의 검색을 위한 기능도 매우 단출한다. Keyspace나 Table등을 과도하게 생성할 경우 Memory Overflow가 발생할 수 있음을 고려해야한다.

![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-06%20%EC%98%A4%ED%9B%84%2010.51.57.png?raw=true)


## 2.구조

Partition Key데이터의 hash값을 기준으로 Data를 분산한다. 처음 각 노드가 Ring에 참여하면 Cassandra의 conf/cassandra.yaml에 정의된 각 설정을 통해 각 노드마다 고유의 Hash 범위를 부여 받는다. Read/Write요청이 오면 Partition Key를 통해서 Hash값을 계산해 데이터가 어느 노드에 있는지 알고 접근할 수 있다. 계산된 Hash값을 Token이라고 부른다.



![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-06%20%EC%98%A4%ED%9B%84%2011.39.58.png?raw=true)
초창기의 Cassandra Data Structure는 **Keyspace > Column Family > Row > Column(Column Name + Column Value)** 형태로 구성되어 있었다.
Keyspace와 Column Family에 대한 정보는 모든 Cassandra Node의 메모리에 저장된다



![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-06%20%EC%98%A4%ED%9B%84%2011.40.02.png?raw=true)
Cassandra 1.2에서는  **Keyspace > Table > Row > Column(Column Name + Column Value)** 로 변경이 되었다.

![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-06%20%EC%98%A4%ED%9B%84%2011.32.18.png?raw=true)

**CQL은 이를 있는 그대로 표현하지 않고 한 단계 추상화해 표현한다.**
위의 CQL Table에서 Row와 Column은 실제 데이터가 저장되는 Cassandra Data Layer의 Row, Column과는 의미가 다르다. CQL에서는 RDBMS의 Tuple, Attribute와 유사한 테이블의 행과 열의 의미와 가깝다.

**Keyspace**
논리적인 Data 저장소

**Table**
다수의 Row로 구성

**Row**
Key-Value로 이루어진 Column으로 구성

**Column**
저장할 데이터

**Virtual Node**
Cassandra에 Data를 CURD를 진행하고. 해당 데이터는 Partition Key로 지정된 Column의 Value가 Row Key가 되고 이 Row Key를 Hashing하여 Token을 계산한 뒤, 해당 Token의 범위에 속한 Node를 찾아 CURD를 진행한다. 각 노드별 Token의 범위가 할당되어 있어야한다.
![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-06%20%EC%98%A4%ED%9B%84%2011.12.34.png?raw=true)

하나의 노드에서 연속한 3개의 Toekn범위에 대한 저장 의무를 가진다.

![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-06%20%EC%98%A4%ED%9B%84%2011.14.13.png?raw=true)

Cassandra 1.2부터 Virtual Node라는 기능이 추가되었다.

하나의 노드의 옵션에서 설정한 만큼 다수개의 가상노드와 가상노드 각각이 정해진 Token범위에 대한 저장 의무를 가진다.
Cassandra 노드안에 가상 노드를 여러개 두고, 아주 잘게 나누어진 token 범위를 가상 노드들에게 할당하여 데이터를 분산하는 개념 Virtual Node을 두어서 데이터를 좀 더 균일하게 분산하고 여러곳에 분산되어 있어 노드의 추가, 제거시 데이터의 이동, 복제, 리밸런싱에 높은 성능향상을 가져다 준다.
conf/cassandra.yaml의 num_tokens의 항목으로 조정할 수 있다.


## 3.용어
##### Partition key
Cassandra에서 Data를 분산 저장하기 위한 Unique한 Key
Partition Key는 Table를 구성할 때 반드시 1개 이상이 지정되어야 하며, 여러 개 지정될 수 있다.
Partition Key가 하나일 경우 해당 Partition Key로 지정된 CQL Column의 Value가 실제 Cassandra Data Layer의 Row Key가 된다.
Partition Key가 여러 개일 경우 :문자와 함께 조합한 값들이 실제 Cassandra Data Layer의 Row Key가 된다.

##### Cluster Key
Cluster Key는 정렬에 대한 기준 역활을 담당한다.

##### Primary Key
Primary Key는 CQL table에서의 각 Row를 각자 Unique 하게 결졍해누즞 기준 역할을 담당한다.
Primary Key는 최소 1개 이상이 Partition Key와 0개 이상의 Cluster Key로 구성된다.

##### Composite Key(=compound key)
1개 이상의 CQL Column들로 이루어진 Primary Key를 Composite Key혹은 Composite key라고 부른다.

##### Composite Partition Key
Composite partition Key는 2개 이상의 다수의 CQL Column으로 이루어진 Partition Key를 의미한다.

## 4.Cassandra Read/Write/Delete/Update
**Write**
![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-08%20%EC%98%A4%EC%A0%84%201.18.43.png?raw=true)
Cassandra에서 최초의 노드를 Coordinator노드라고 부른다. Coordinator노드는 Row Key를 Hashing해 어느 노드들에 데이터를 Write해야하는지 확인을 한다. 그리고 Consistency Level에 따라서 몇 개의 노드에 Write를 해야하는지 참고해 현재 데이터를 Write해야 할 노드들의 status가 정상인지를 확인한다. 특정 노드가 정상적이지 않다면 Consistency Level에 따라 hint hand off라는 로컬 임시 저장공강에 Write할 데이터를 저장한다. 정상적으로 돌아오면 다시 Coordinator노드가 data를 Write해주기 위해서이다.

![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-08%20%EC%98%A4%EC%A0%84%201.23.45.png?raw=true)
데이터를 저장하게 될 노드는 Write요청이 오면 혹시 모를 장애에 대비해 Commitlog라고 불리는 로컬 디스크의 파일에 기록을 남긴다. 그리고 Mem Table이라는 이름의 메모리 저장공에 데이터를 Wirte한 뒤, 성공 메시지를 돌려줘 Wirte의 요청을 마무리한다.
Mem Table에 데이터가 충분히 쌓이면 디스크 버전의 Mem Table인 "SSTable"에 데이터를 Flush한다. SSTable은 Immutable하며, Sequential하다는 특징을 가지고 있으며 Cassandra는 이러한 다수의  SSTable을 Compaction해 데이터를 관리한다.

**Read**
![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-08%20%EC%98%A4%EC%A0%84%201.26.55.png?raw=true)

Read요청이 오면 Coordinator 노드는 해당 요청의  Row Key를 Hashing해 접근해야할 노드의 위치를 파악한 뒤, Consistency Level를 체크해 몇 개의 Replication을 확인해야 할지 결정한다. 그리고 Coordinator노드는 데이터가 있는 가장 가까운 노드에 Data Request를 요청하고, 그다음 가까운 노드들에는 Data Digest Reqeust를 요청한다. 이러게 가져온 데이터를 비교해 정보가 일치하지 않으면 일치 하지 않은 데이터들의 노드로부터 Full Data를 가져와 그중 가장 최신 데이터를 사용자에게 돌려준다. 그리고 최신 데이터를 기준으로 나머지 노드들의 데이터를 수리한다.

![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-08%20%EC%98%A4%EC%A0%84%201.27.03.png?raw=true)

1.Mem Table를 확인한다.

2.Bloom Filter를 확인한다.
 - Bloom Filter란 긍정 오류는 발생할 수 있지만, 부정 오류는 발생하지 않는 확률적인 자료구조, 없는걸 있다고 거짓말 할 수는 있지만 있는걸 없다고 하지 않는다.

3.Index를 확인한다.
  - 메모리에 저장되어 있는 Summery Index를 통해서 디스크에 저장되어 있는 원본 Index를 확인하여 SSTable 내의 DATA 위치에 대한 offset을 알게된다.

**Delete**
 Cassandra는 바로 Delete를 하지 않고 모든 데이터는 Tombstone이라는 marker가 존재하고, 해당 Tombstone에 마킹을 하고 주기적인 GC, SSTable의 Compaction에 의해서 삭제가 된다.

**Update**
Cassandras는 SSTable이 Immutable하기 때문에 Update는 Delete -> Insert 작업으로 이루어 진다.


## 5.주의 사항
**Delete**
![cassandra](https://github.com/namgunghyeon/wiki/blob/master/images/cassandra/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-08%20%EC%98%A4%EC%A0%84%201.52.02.png?raw=true)

Read성능
MemTable이나 SSTable의 데이터를 삭제 했더라도 Tombstone이 Marking되어있을 뿐 실제로는 Compaction이 발생전 까지 디스크에 존재 데이터를 Read할 때 순서대로 읽기 때문에 삭제 마킹이 되어도 읽고 지나간다.

**Secondary Index**
Range쿼리을 사용할 수 없음.

**Memory Orverflow**
Cassandra는 모든 Keyspace와 Table에 대한 Metadata를 JVM 메모리에 올려 놓고 사용하고 있고 이것은 분산 되지 않고 Ring를 구성하는 모든 노드가 동이랗게 가지고 있는 데이터이다. 많은 Keyspace와 Table를 사용할 경우 메모리를 급격하게 소진할 수 있다.

## 6.설치
자바 설치
```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-set-default
java -version
```

Cassandra 설치
```bash
echo "deb http://www.apache.org/dist/cassandra/debian 22x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
echo "deb-src http://www.apache.org/dist/cassandra/debian 22x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list

gpg --keyserver pgp.mit.edu --recv-keys F758CE318D77295D
gpg --export --armor F758CE318D77295D | sudo apt-key add -

gpg --keyserver pgp.mit.edu --recv-keys 2B5C1B00
gpg --export --armor 2B5C1B00 | sudo apt-key add -

gpg --keyserver pgp.mit.edu --recv-keys 0353B12C
gpg --export --armor 0353B12C | sudo apt-key add -

sudo apt-get update

sudo apt-get install cassandra
```

Cassandra 연결
```bash
상태 확인
sudo nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  127.0.0.1  97.11 KB   256          100.0%            77e8d23b-3e0d-45da-8700-e631113a40b9  rack1


접속
cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 2.2.8 | CQL spec 3.3.1 | Native protocol v4]
Use HELP for help.
cqlsh> exit

```

Host 설정
```bash
vi /etc/hosts
192.168.56.119 cassandra01
192.168.56.200 cassandra02
```

Cassandra Cluster
```bash
sudo service cassandra stop
sudo rm -rf /var/lib/cassandra/data/system/*

sudo vi /etc/cassandra/cassandra.yaml

cluster_name: 'CassandraDOCluster'

seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "your_server_ip,your_server_ip_2,...your_server_ip_n"

listen_address: your_server_ip
rpc_address: your_server_ip
endpoint_snitch: GossipingPropertyFileSnitch

파일 맨 마지막에 추가
auto_bootstrap: false

#  - GossipingPropertyFileSnitch
#    This should be your go-to snitch for production use.  The rack
#    and datacenter for the local node are defined in
#    cassandra-rackdc.properties and propagated to other nodes via
#    gossip.  If cassandra-topology.properties exists, it is used as a

sudo service cassandra start
sudo nodetool status
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address         Load       Tokens       Owns (effective)  Host ID                               Rack
UN  192.168.56.200  159.65 KB  256          100.0%            87da18e8-a977-4316-8261-048972cbfa16  rack1
UN  192.168.56.119  149.7 KB   256          100.0%            77e8d23b-3e0d-45da-8700-e631113a40b9  rack1

```

Cassandra 접속
```bash
cqlsh cassandra02 9042
Connected to Test Cluster at cassandra02:9042.
[cqlsh 5.0.1 | Cassandra 2.2.8 | CQL spec 3.3.1 | Native protocol v4]
Use HELP for help.
cqlsh> exit

cqlsh cassandra01 9042
Connected to Test Cluster at cassandra01:9042.
[cqlsh 5.0.1 | Cassandra 2.2.8 | CQL spec 3.3.1 | Native protocol v4]
Use HELP for help.
cqlsh>
```

간단하게 Cassandra를 설치할 수 있다.

**출처:
http://meetup.toast.com/posts/58
https://www.digitalocean.com/community/tutorials/how-to-run-a-multi-node-cluster-database-with-cassandra-on-ubuntu-14-04https://www.digitalocean.com/community/tutorials/how-to-install-cassandra-and-run-a-single-node-cluster-on-ubuntu-14-04**
