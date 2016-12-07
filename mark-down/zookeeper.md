# Zookeeper 정리
## 1.Zookeeper란

분산 어플리케이션을 위한 분산 관리서비스(코디네이션 서비스 시스템)
Zookeeper는 Hadoop의 서브 프로젝트로 시작되었다. Hadoop에서 돌아가는 HDFS(Hadoop Distributed File System)이나 HBase같은 어플리케이션의 분산 처리를 제어할 때 Zookeeper를 사용한다.
코디네이션 서비스는 분산 시스템 내에서 중요한 상태 정보나 설정 정보등을 유지하기 때문에 서비스의 장애는 전체 시스템의 장애를 유발하기 때문에, 이중화등을 통하여 고가용성을 제공해야 한다. Zookeeper는 이러한 특성을 잘 제공하고 있어, 유명한 분산 솔루션에 많이 사용되고 있다.

![zookeeper](https://raw.githubusercontent.com/namgunghyeon/wiki/8e2fa9bebaffb848c7018f399a1a4533fd502990/images/zookeeper/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-14%20%EC%98%A4%EC%A0%84%202.53.58.png)

## 2.구조
디렉토리 구조 기반으로 znode라는 데이터 저장 객체를 제공하고 Key-Value 이 객체에 데이타를 넣고 빼는 기능함 제공한다. 디렉토리 형식을 사용하기 때문에 데이터를 계층화된 구조로 저장하기 용이하다.


**path** : 주키퍼에서 노드를 구분하는 식별자는 파일 시스템과 동일하게 / 이다.
**znode**: 네임 스페이스의 각 노드를 znode라고 부른다.
즉 /, /node1, /node1/config등을 znode라고 부른다.
**Persistent Node**:
노드에 데이타를 저장하면 삭제 하지 않는 이상 삭제 되지 않고 영구 저장된다.
**Ephemeral Node**:
노드를 생성한 클라이언트의 세션이 연결되어 있을 경우에만 유효하다. 클라이언트 연결이 끊어지는 순간 삭제가 된다. 이를 통해서 클라이언트가 연결이 되었는지 아닌지
판단하는데 사용할 수 있다.

**Sequence Node**:
노드를 생성할때 자동으로 sequence번호가 붙는 노드이다. 주로 분산락을 구현하는데 이용된다.

Zookeeper는 데이터를 디스크에 영구 저장하기 하지만, 빠른 처리를 위해 모든 트리 노드를 메모리에 올려 놓고 처리한다. 대규모 데이터를 처리하기엔 무리가 있다.따라서 Memcached나 Redis와 같은 캐시 서버와는 다른 용도로 쓰인다.

Zookeeper의 여러 기능 가운데 가장 활용도가 높은 부분은 데이터 변경을 감시하여 콜백을 실행하는 감시자 Watch이다. 감시자를 트리 내의 특정 노드에 등록하면 별도로 테이터를 폴링을 하지 않아도 노드 변경 사항을 전달 받을 수 있다.

**Watcher**
Watch 기능은 ZooKeeper 클라이언트가 특정 znode에 watch를 걸어 놓으면, 해당 znode가 변경이 되었을때, 클라이언트로 callback호출을 날려서 클라이언트에 해당 znode가 변경되었음을 알려준다.


**주키퍼의 보장 내용**
순차적인 일관성: 클라이언트로부터의 업데이트는 순서대로 적용된다.
원자성 : 업데이트는 성공하거나 실패한다. 그 외 부분은 없다.
단일 시스템 이미지 : 클라이언트는 서버와 연결이 되어 있는 동안 서비스를 같은 시각으로 볼 수 있다.
신뢰성 : 한번 업데이트가 적용되면, 이것은 클라이언트가 업데이트를 덮어 쓰기 전까지 지속된다.
적시성 : 시스템의 클라이언트 시각적 관점은 특정 시간 내에 처리 될 수 있도록 보장한다.

![zookeeper](https://raw.githubusercontent.com/namgunghyeon/wiki/8e2fa9bebaffb848c7018f399a1a4533fd502990/images/zookeeper/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-14%20%EC%98%A4%EC%A0%84%202.54.04.png)

**Zookeeper 앙상블**
분산 작업을 제어하기 위해 사용되는 만큼 Zookeeper 서버가 중단되면 ZooKeeper에 의존하는 모든 서버가 영향을 받는다. 따라서 Zookeeper는 최대한 정상 동작을 보장해야 한다. Zookeeper 서버를 클러스터링해 고가용성을 지원하도록 설정한다. Zookeeper클러스터를 앙상블이라고 부른다.

앙상블로 묶인 ZooKeeper 서버중 한 대는 쓰기 명령을 총괄하는 리더 역활을 수행하고, 나머지는 팔로어 역활을 수행한다. 클라이언트에서 Zookeeper 앙상블에 연결할 때는 커넥션 문자열에 앙상블을 구성하는 Zookeeper서버 주소를 다수 포함할 수 있다. 클라이언트는 커넥션 문자열이 포함된 Zookeeper 주소중 하나에 접근하여 세션을 맺는다.

클라이언트가 전달한 읽기 명령은 현재 연결된 Zookeeper 서버에서 바로 변환한다. 이에 비해 쓰기 명령은 앙상블 중 리더 역할을 수행하는 Zookeeper 서버로 전달되며, 리더 Zookeeper는 모든 팔로어 Zookeeper에게 해당 쓰기를 수행할 수 있는지 질의한다. 팔로어중 과반수 팔로어로부터 쓸 수 있다는 응답을 받으면 리더는 팔로어에게 데이터를 쓰도록 지시한다.

## 3.설치
VirtaulBox로 3대 설치
![zookeeper](https://raw.githubusercontent.com/namgunghyeon/wiki/8e2fa9bebaffb848c7018f399a1a4533fd502990/images/zookeeper/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-14%20%EC%98%A4%EC%A0%84%202.54.11.png)

다운
http://zookeeper.apache.org, http://www.apache.org/dyn/closer.cgi/zookeeper/, http://apache.mirror.cdnetworks.com/zookeeper/stable/  
현재 안정버전 : zookeeper-3.4.9.tar.gz

Host설정
```
vi /etc/hosts
192.168.56.112 zookeeper01
192.168.56.113 zookeeper02
192.168.56.114 zookeeper03
```
자바 설치
```
add-apt-repository ppa:webupd8team/java
apt-get update
apt-get install oracle-java8-installer
apt-get install oracle-java8-set-default
```
ZooKeeper 디렉토리 생성
```
mkdir /home/data
mkdir /home/data/zookeeper
```
Zookeeper 설정 변경
```
주키퍼가 설치된 곳
cd /home/zookeeper1/zookeeper-3.4.9/conf
cp zoo_sample.cfg zoo.cfg

vi zoo.cfg
dataDir=/home/data/zookeeper
server.1=zookeeper01:2888:3888
server.2=zookeeper02:2888:3888
server.3=zookeeper03:2888:3888
```
Myid 생성
```
/home/data/zookeeper 디렉토리에 서버별로 생성
zoopeeker01
vi /home/data/zookeeper/myid
1
zoopeeker02
vi /home/data/zookeeper/myid
2
zoopeeker03
vi /home/data/zookeeper/myid
3
```
주키퍼 실행
```
/home/zookeeper1/zookeeper-3.4.9/bin
bash zkServer.sh start
```
주키퍼 중지
```
/home/zookeeper1/zookeeper-3.4.9/bin
bash zkServer.sh stop
```
에러 확인
```
/home/zookeeper1/zookeeper-3.4.9/bin
vi  zookeeper.out
```
주키퍼 접속
```
/home/zookeeper1/zookeeper-3.4.9/bin
bash zkCli.sh -server zookeeper01:2181
```
## 4.TEST
Ephemeral Node모들 통해서 새로운 클라이언트가 접속하고 이를 감지하는 Watcher
[python_zookeeper_watcher](https://github.com/namgunghyeon/python_zookeeper/tree/master/ephemeral)

## 4.TEST_2
Zookeeper Lock Test
[python_zookeeper_lock](https://github.com/namgunghyeon/python_zookeeper/tree/master/lock)

***출처:
http://bcho.tistory.com/search/Kafka
http://jdm.kr/blog/125
http://d2.naver.com/helloworld/583580
http://advent.perl.kr/2012/2012-12-24.html
https://zookeeper.apache.org/
http://www.joinc.co.kr/w/man/12/zookeeper/Watcher***
