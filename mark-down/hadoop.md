#Hadoop정리
## 1.Hadoop란
대용량 데이터를 분산 처리할 수 있는 자바기반의 오픈소스 프레임워크

블록 구조의 파일 시스템
HDFS에 저장하는 파일은 특정 크기의 블록으로 나누어저 분산된 서버에 저장된다. 블록크기는 기본적으로 64MB(현재는 128MB) 설정돼 있으며, 여러재의 블록은 동일한 서버에 저장되는 것이 아니라 여러 서버에 나눠서 저장된다.

### 네임노드와 데이터 노드
HDFS는 마스터 - 슬레이브 아키텍쳐로 마스터서버는 네임노드이며, 슬레이브 서버는 데이터 노드이다.
![hadoop](https://github.com/namgunghyeon/wiki/blob/master/images/hadoop/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-03%20%EC%98%A4%EC%A0%84%2012.19.53.png?raw=true)
![hadoop](https://github.com/namgunghyeon/wiki/blob/master/images/hadoop/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-03%20%EC%98%A4%EC%A0%84%2012.04.01.png?raw=true)
[hadoop architecture](http://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)


### 네임 노드
***-메타 데이터 관리***

네임노드는 파일 시스템을 유지하기 위한 메타데이터를 관리한다. 메타데이터는 파일 시스템 이미지와 파일에 대한 블록 맴핑 정보로 구성된다. 네임노드는 클라이언트에게 빠르게 응답할 수 있게 메모리에 전체 메타데이터를 로딩해서 관리한다.

***-데이터 노드 모니터링***

데이터 노드는 네임노드에게 3초마다 하트비트 메시지를 전송한다. 하트비트는 데이터노드 상태정보와 데이터노드에 저장돼 있는 블록의 목록으로 구성된다. 네임노드는 하트 비트를 이용해 데이터 노드의 실행 상태와 용량을 모니터링한다.

***-블록 관리***

네임 노드는 다양한 방법으로 블록을 관리한다. 네임노드는 장애가 발생한 데이터노드를 발견하면 해당 데이터 노드의 블록을 새로운 데이터노드로 복제한다. 또한 용량이 부족한 데이터노드가 있다면 용량에 여유가 있는 데이터노드 블록을 이동시킨다. 복제본 수도 관리한다. 만약 복제본 수와 일치하지 않는 블록이 발견될 경우 추가로 블록을 복제하거나 삭제를 해준다.


***-클라이언트 요청 접수***

클라이언트가 HDFS에 접속하려면 받드시 네임노드에 먼저 접속해야한다.


### 데이터 노드
데이터 노드는 클라이어트가 HDFS에 저장하는 파일을 로컬 디스크에 유지한다. 로컬 디스크에 저장되는 파일은 두 종류로 구성된다. 첫 번째 파일은 실제 데이터가 저장돼 있는 로우 데이터이며, 두 번째 파일은 체크섬이나 파일 생성 일자와 같은 메타데이터가 설정돼 있는 파일입니다.

***-파일 저장 요청***

![hadoop](https://github.com/namgunghyeon/wiki/blob/master/images/hadoop/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-03%20%EC%98%A4%EC%A0%84%2012.08.52.png?raw=true)

클라이언트가 HDFS에 파일을 저장하는 경우 파일을 저장하기 위한 스트림을 생성해야한다. 클라이언트가 네임노드와 통신 과정을 통해 파일 저장용 스트림 객체를 생성한다.
```
다른 책에서는 DataNode에 Write가 완료되면 NameNode에 저장 결과 알림을 전달한다고 되어 있다.
```

***-패킷 전송***

클라이언트가 네임노드에게서 파일 제어권을 얻게 되면 파일 저장을 진행한다. 이때 클라이언트는 파일을 네임노드에 전송하지 않고 각 데이터노드에게 전송한다. 그리고 저장할 파일은 패킷 단위로 나눠서 전송한다.

***-파일 닫기***

***-파일 조회 요청***
![hadoop](https://github.com/namgunghyeon/wiki/blob/master/images/hadoop/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-03%20%EC%98%A4%EC%A0%84%2012.09.57.png?raw=true)

#### 보조 네임노드
![hadoop](https://github.com/namgunghyeon/wiki/blob/master/images/hadoop/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-03%20%EC%98%A4%EC%A0%84%2012.10.17.png?raw=true)

네임노드는 메타 데이터를 메모리에 처리한다. 하지만 메모리에만 데이터를 유지할 경우 서버가 재부팅될 경우 모든 메타데이터가 유실 될 수 있다. HDFS는 이러한 문제를 극복하기 위해서 editlog와 fsimage라는 두개의 파일을 생성한다.

editlog는 HDFS의 모든 변경 이력을 저장한다. HDFS는 클라이언트가 파일을 저장하거나, 삭제허간, 혹은 파일을 이동하는 경우 editlog와 메모리에 로딩돼 있는 메타데이터에 기록한다. fsimage는 메모리에 저장된 메타데이터의 파일 시스템 이미지를 저장한 파일이다.

    1.네임노드가 구동되면 로컬에 저장된 fsimage와 editlog를 조회한다.
    2.메모리에 fsimage를 로딩해 파일 시스템 이미지를 생성한다.
    3.메모리에 로딩된 파일 시스템 이미지를 editlog에 기록된 변경 이력을 적용한다.
    4.메모리에 로딩된 파일 시스템 이미지를 이용해 fsimage파일을 갱신한다.
    5.editlog를 초기화한다.
    6.데이터노드가 전송한 블록리포트를 메모리에 로딩된 파일 시스템 이미지에 적용한다.

보조 네임노드는 주기적으로 네임노드의 fsimage를 갱신하는 역활을하며, 이러한 작업을 체크포인트라고한다. 흔히 보조 네임노드를 체크포인팅 서버라고 표현한다.


## 2.설치
java
```
# add-apt-repository ppa:webupd8team/java
# apt-get update
# apt-get install oracle-java8-installer
# apt-get install oracle-java8-set-default
java version "1.8.0_111"
```

Hadoop 계정 생성
```
adduser hadoop
su - hadoop
```

SSH Key등록
```
한 노드에서 다른 노드를 접속할 때 패스워드를 입력하지 않도록 하기 위해서 모든 노드에 SSH key를 등록한다.
ssh-keygen -t rsa 키 생성
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  공개키 등록
scp로 모든 도느에 .ssh경로 아래에 id_rsa.pub, authorized_keys 파일을 복사한다.
ssh로 다른 서버에 접속할 때 패스워드 없이 접속이 가능해야 한다.
```

Hosts설정
```
마스터, 호스트 주소를 alias설정
sudo vi /etc/hosts
192.168.56.107 hadoop-master-1
192.168.56.108 hadoop-master-2
192.168.56.109 hadoop-slave-01
192.168.56.110 hadoop-slave-02
192.168.56.111 hadoop-slave-03
```

Hadoop설치
```
wget http://apache.mirror.cdnetworks.com/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
tar -xvf hadoop-2.7.3.tar.gz

작업 디렉토리 설정(모든 서버에 설정이 되어야 함)
adduser hadoop 하둡 계정 생성(모든 서버에 설정이 되어야 함)
sudo mkdir -p /data/hadoop/tmp
sudo mkdir -p /data/hadoop/dfs/name
sudo mkdir -p /data/hadoop/dfs/data
sudo chown -R hadoop:hadoop /data/hadoop
```

Hadoop설정
core-site
```
$HADOOP_HOME/etc/hadoop/core-site.xml 수정
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop-master-1:9000</value>
  </property>
  <property>
   <name>hadoop.tmp.dir</name>
   <value>/data/hadoop/tmp</value>
  </property>
</configuration>

fs.defaultFS
 HDFS의 기본 이름을 의미하며, URI형태로 사용된다. 데이터 노드는 여러 작업을 진행하기 위해 받느딧 네임도의 주소를 알고 있어야 한다. 네임노드로 하트비트나 블록 리포트를 보낼 때 바로 이 값을 참조해 네임노드를 호출한다.

hadoop.tmp.dir
하둡에서 발생하는 임시 데이터를 저장하기 위한 공간. 기본적으로 root 디렉토리의 하위 디렉토리인 tmp디렉토리에서 데이터를 생성한다.
```
hdfs-site
```
$HADOOP_HOME/etc/hadoop/hdfs-site.xml 수정
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/data/hadoop/dfs/name</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/data/hadoop/dfs/data</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
</configuration>

dfs.replication
HDFS의 저장될 데이터의 복제본 개수를 의미한다.
```

mapred-site
```
$HADOOP_HOME/etc/hadoop/mapred-site.xml 수정
cp mapred-site.xml.template mapred-site.xml
<configuration>
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
  </property>
  <property>
    <name>mapred.local.dir</name>
    <value>/data/hadoop/hdfs/mapred</value>
  </property>
  <property>
    <name>mapred.system.dir</name>
    <value>/data/hadoop/hdfs/mapred</value>
  </property>
</configuration>

```

yarn-site.xm
```
$HADOOP_HOME/etc/hadoop/yarn-site.xml 수정
<configuration>
<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>hadoop-master-1:8025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>hadoop-master-1:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>hadoop-master-1:8035</value>
  </property>
</configuration>
```

기타
```
$HADOOP_HOME/etc/hadoop/masters
hadoop-master-1
$HADOOP_HOME/etc/hadoop/slaves
hadoop-slave-01
hadoop-slave-02
hadoop-slave-03
$HADOOP_HOME/etc/hadoop/hadoop-env.sh 수정
JAVA_HOME = /usr/lib/jvm/java-8-oracle
rsync 모든 노드에 복사
rsync -avz ~/hadoop-2.7.3 hadoop@hadoop-master-2:/home/hadoop/
rsync -avz ~/hadoop-2.7.3 hadoop@hadoop-slave-01:/home/hadoop/
rsync -avz ~/hadoop-2.7.3 hadoop@hadoop-slave-02:/home/hadoop/
rsync -avz ~/hadoop-2.7.3 hadoop@hadoop-slave-03:/home/hadoop/
```

하둡 실행
```
HDFS포멧
/home/hadoop/hadoop-2.7.3/bin
bash hadoop namenode -format

sbin/start-dfs.sh
sbin/start-yarn.sh

```
http://192.168.56.107:50070
아래 이미지 처럼 Live Nodes가 NameNode에 맞게 나와야 정상
![hadoop](https://github.com/namgunghyeon/wiki/blob/master/images/hadoop/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-23%20%EC%98%A4%EC%A0%84%204.11.37.png?raw=true)


네임노드가 정상적으로 UI화면에 나오지 않을 때
```
/etc/hosts에 아래와 같이 127.0.1.1 있을 경우 삭제하고 네임노드를 다시 포멧하고 재시작
127.0.1.1 hadoop-master-1

네임노드를 포멧할 때 Re-format filesystem in Storage Directory /data/hadoop/dfs name ? (Y or N) Y

http://mazdah.tistory.com/606
http://www.bicdata.com/bbs/board.php?bo_table=develop_hadoop&wr_id=597
http://gh0stsp1der.tistory.com/66

```

## 3.맵리듀스(요즘 사용하지 않지만 개념만 알고가기)
맵리듀스 프로그래밍 모델은 맵과 리듀스라는 두 가지 단계로 데이터를 처리한다. 맵을 입력 파일을 한 줄씩 읽어서 데이터를 변형하며, 리듀스는 맵의 결과 데이터를 집계한다.

작성중

## 4.Hadoop 명령어
기본 형태
~/hadoop-2.7.3/bin
hadoop fs -<명령어>

```
bash hadoop fs -ls /
drwxr-xr-x   - tajo supergroup          0 2016-10-22 21:53 /movielens
drwxr-xr-x   - tajo supergroup          0 2016-10-22 21:40 /tajo
drwxr-xr-x   - tajo supergroup          0 2016-10-22 21:40 /tmp
```

```
bash hadoop fs -count /
          20            9           24887031 /
```
```
bash hadoop fs -du /
24765439  /movielens
104905    /tajo
16687     /tmp
```

```
하둡에서 자주 사용하는 명령어는 다음과 같다.
* 폴더의 용량을 확인할 때 count 를 사용
* 파일의 내용을 확인할 때는 cat 보다는 text를 사용하면 더 좋다. 파일 타입을 알아서 판단하기 때문

hadoop fs -cat [경로]
    - 경로의 파일을 읽어서 보여줌
    - 리눅스 cat 명령과 동리함

hadoop fs -count [경로]
    - 경로상의 폴더, 파일, 파일사이즈를 보여줌

hadoop fs -cp [소스 경로] [복사 경로]
    - hdfs 상에서 파일 복사

hadoop fs -df /user/hadoop
    - 디스크 공간 확인

hadoop fs -du /user/hadoop
    - 파일별 사이즈 확인

hadoop fs -dus /user/hadoop
    - 폴더의 사이즈 확인

hadoop fs -get [소스 경로] [로컬 경로]
    - hdfs 의 파일 로컬로 다운로드

hadoop fs -ls [소스 경로]
    - 파일 목록 확인

hadoop fs -mkdir [생성 폴더 경로]
    - 폴더 생성

hadoop fs -put [로컬 경로] [소스 경로]
    - 로컬의 파일 hdfs 상으로 복사

hadoop fs -rm [소스 경로]
    - 파일 삭제, 폴더는 삭제 안됨

hadoop fs -rmr [소스 경로]
    - 폴더 삭제

hadoop fs -setrep [값] [소스 경로]
    - hdfs 의 replication 값 수정

hadoop fs -text [소스 경로]
    - 파일의 정보를 확인하여 텍스트로 반환
    - gz, lzo 같은 형식을 확인후 반환해줌
```
참고
https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#count


## 5.Hadoop 2.0
### hadoop 1의 문제점
- **jobtracker의 단일고장점화(NameNode)**
  - 하둡1의 jobtracker는 모든 맵리듀스 job의 실행 요청을 받아, 전체 job의 스케쥴링과 리소를 관리
  - 클라이언트가 맵리듀스 job을 실행하기 위해서는 반드시 jobtracker가 실행 중이어야 함
  - Jobtracker는 단일고장점(SPoF: Single Point of Failure)이 됨

- **Memery ISSUE**
  - Jobtracker는 메모리상에 전체 job의 실행 정보를 유지, 관리에 활용하여 많은 메모리 용량 소모
  - 만일 메모리의 용량이 부족하면 job의 상태를 모니터링 할 수 없고, 새로운 job을 실행할 수 도 없음

- **비효율직인 리소스 관리**
 - 하둡 1 맵리듀스 프레임워크에서는 맵퍼와 리듀서를 따로 설정
   - Mapper는 모두 동작하는데 Reducer는 아무 작업도 하지 않고 있을 때
     - 전체 클러스터 입장에서는 리소스가 낭비됨

### Hadoop2로 진화
Yarn 하둡2의 핵심 시스템


Yarn의 구성
- **Resource Manager**
  - golbal scheduler, 즉 마스터
  - 전체 클러스터에서 가용한 모든 시스템 자원을 관리
  - Application이 리소스를 요청하면 이를 적절히 분해하고 그 사용 상태를 모니터링
  - Scheduler, Application manager, Resource tracker

- **Node Manager**
  - 슬레이브 서버의 시스템 자원을 나타내는 컨테이너들을 실행하고 이들을 모니터링
  - 컨테이너들은 resource manager의 요청에 따라서 실행되며 하나의 슬레이브 서버에는 여러 컨테이너가 실행될 수 있음
  - 하나의 Application을 관리하는 application master
    - 클라이언트가 yarn에 application 실행을 요청하면
      - application하나당 하나의 application master를 할당하며 이는 컨테이너에서 실행됨
    - application master는 application에 필요한 리소스를 scheduling하고 node manager에 필요한 컨테이너를 요청함
  - applcation mater, Container

```
Name Node: name node가 정상적으로 동작하지 않을 경우 모든 클라이언트가 HDFS에 접근할 수 없다. name node에 HDFS의 디렉토리 구조와 파일 위치가 모두 저장되어 있기 때문에, 이정보가 유실될 경우 데이터블록에 접근할 통로가 없어지게 된다. edit log에 문제가 발생해도 이와 비슷한 상황이 발생한다.

Name Node HA: 네임노드의 이중화(Active/Standby) Shared edits: 여러 서버(저널 노드)에 복제 저장. 저널 도느는 최소 3개 이상, 홀수개로, 전체 저널 노드 중 반드시 절반 이상이 실행되어 있어야함

ZKFC(zookeeper failover controller) : 네임노드 상태 모니터링, active 네임노드에 장애 발생시 standby 네임노드를 active 네임노드로 전환

HDFS federation : 기존 HDFS가 단일 네임노드에 기능이 집중되는 것을 막기 위해 제안된 시스템. HDFS는 다수의 네임노드로 구성되며 각 네임노드는 독립된 네임스페이스를 관리한다. 추가적으로 HDFS는 동합된 네임스테이즈를 제공하지 않으므로 전체 네임노드의 디렉토리가 설정돼 있는 마운트 테이블을 생성해 통합된 네임스테이스를 참조할 수 있다.
```




***출처:
http://crazia.tistory.com/entry/HADOOP-%ED%95%98%EB%91%A1-Hadoop-260-%EC%84%A4%EC%B9%98#recentTrackback
http://inthound.blogspot.kr/2015/04/how-to-install-hadoop-260-in-ubuntu.html
http://datamining.dongguk.ac.kr/hadoop/
http://rocksea.tistory.com/282
http://www.slideshare.net/Raze2dust/hadooparchitecture-110910130636phpapp01
http://blog.bigdataweek.com/2016/08/01/hadoop-important-handling-big-data/***
