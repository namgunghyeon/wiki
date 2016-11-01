# Spark정리
## 1.Spark란
Apach Spark는 빠르고 General Purpose Cluster Computing System입니다. 스팍은 범용 분산 플랫폼. 하둡과 같이 MapReduce만 돌리는 것은 아니고, Storm과 같이 스트리밍만 처리하는게 아니라, 분산된 여러대의 노드에서 연산할 수 있게 해주는 범용 분산 클러스터링 플랫폼으로, 위에 MapRduce, 스트리밍 처리등의 모듈을 올려서 수행할 수 있다. 하둡이 MR작업을 디스크 기반으로 수행하기 때문에 느려지는 성능을 메모리 기반으로 옮겨서 고속화 하고자  출발했다.
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.05.04.png?raw=true)

## 2.구조
스팍은 Driver Program으로 Driver Program은 여러개의 병렬 작업으로 나누어 Spark의 Worker Node에 있는 Executor에서 실행된다.

![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.05.09.png?raw=true)

**Cluster Manager Type**:
Standalone, Apache Mesos, Hadoop YARN


**Driver Program**
main 함수를 가지고 있는 프로세스, spark-sumit을 통해서 구현한 코드를 제출, 구현한 코드에서는 SparkContext라는 객체를 생성하고, RDD를 생성하며 제출한 application을 Task라고 불리는 실제 수행 단위로 변환 task를 묶어 Worker Node의 Executor로 전달을 한다. Executor는 받은 Task를 RDD에 저장하고 처리를 한다.(spark-sumit이 워커 노드의 정보를 알고 모든 워크 노드에게 작업을 시키는 건지?, 클러스터 매니저로부터 워커 노드 정보를 받아서 전달?)


**Worker Node**
클러스터에 있는 executor를 포함한 실제 작업을 하는 노드

**SparkContext**
Driver Program에 의해서 생성되고, Cluster manager(Resource를 할당하는 역활)와 연결된다.


**Executor**
computation과 data를 저장하는 역할을 하는 process로, application과 lifecycle과 동일하게 수행된다. executor가 오류가 나면 대체 executor job을 할당 한다. Executor는 multi threads에서  tasks를 수행하고 수행 결과를 Driver Program으로 전송 및 tasks를 스케쥴링하는 역할을 한다.

![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.05.14.png?raw=true)

**Spark Application 실행 동작 순서**
```
1.사용자가 spark-submit을 사용해 작성한 Application을 실행
2.spark-submit은 Driver Program을 실행하여 main() 메소드를 호출
3.Driver에서 생성된 SparkContext는 Cluster Manager로 부터 Executor 실행을 위한 리소스를 요청
4.Cluster Manager는 Executor를 실행
5.Driver Program은 Application을 Task단위로 나누어 Executor에게 전송
6.Executor는 Task를 실행
7.Executor는 Application이 종료되면, 결과를 Driver Program에게 전달, Cluster Manager에게 리소를 반납.
```


## 3.RDD(Resilient Distributed DataSet)
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.05.21.png?raw=true)
Resilient Distributed Datasets의 특성은 immutable, partitioned collections of records의 특징이 있다.
수정 불가능한 객체로 생성하는 방법은 Storage에서 데이터를 불러와 RDD로 변환하거나 RDD에서 RDD로만 가능하다.
**immutable이기 때문에 수정이 불가능해, read-only로만 사용되고, 생성되는 과정을 기록해 놓은 linege를 통해서 RDD객체를 다시 생성할 수 있다. 이렇게하면 fault-tolerant의 문제를 해결할 수 있다.**

linegage는 계보라는 뜻을 갖고 있고, DAG(directed acyclic graph)로 디자인 되어 있다. 데이터를 로딩하고, 일련의 과정을 기록하고 있고, 이렇게 기록된 과정은 추후 fault-tolerant의 문제가 발생할 때 내가 생성해 놓은 RDD의 이전 lineage를 보고 생성하면 되기 때문에 빠른 복구 가능하다.

RDD는 여러 분산 노드에 걸쳐서 저장되는 변경이 불가능한 데이터의 집합으로 각각의 RDD는 여러개의 파티션으로 분리가 된다.
RDD는 변경이 불가능해, 변경하려면 새로운 데이터 셋을 생성해야 한다.
두 가지 오퍼레이션만 지원한다.

**Transformation** : 기존의 RDD 데이타를 변경하여 새로운 RDD 데이타를 생성해내는 것. filter와 같은 특정한 데이타만 뽑아 내거나 map 함수 처럼, 데이타를 분산 배치 하는 것 등을 들 수 있다.

**Action** : RDD 값을 기반으로 무언가를 계산해서 결과를 생성해 내는것으로 Count()와 같은 Operation들을 들 수 있다. RDD의 데이타 로딩 방식을 Lazy 로딩 컨셉을 사용하는데, 예를 들어 sc.textFile(‘파일')로 파일을 로딩하더라도 실제로 로딩이 되지 않는다. 파일이 로딩되서 메모리에 올라가는 시점은 action을 이용해서 개선할 당시만 올라간다. RDD를 action을 만나기 전까지 transformation을 처리하지 않고 RDD는 데이터를 갖고 있기 보다는 reference를 가지고 있다.

**Data Partitioning**
분산 프로그램에서 동신 비용이 매우커 네트워크 부하를 줄이는 것이 중요. 파티셔닝은 네트워크 부하를 효율적으로 줄이는 방법
어떤 키의 모음들의 임의의 노드에 함께 모여 있는 것을 보장
키 중심의 연산에서 데이터가 여러번 재 사용될때만 의미가 있음

**Cache**
Transformation과 Action의 Operation을 통해 데이터를 처리하고. 메모리에 로딩하는 작업을 진행 하지만 Action의 operation을 사용하고 나면, 데이터는 결과를 반환 하고 메모리에서 사라지게 된다. action을 할 때마다 데이터를 처리하고 로딩하고 반환하는 작업을 반복적으로 진행하게 된다. RDD를 메모리에 올린 상태에서 재활용해서 사용할 수 있는 방법은 Persist을 사용한다. 이를 통해서 메모리에 상주 시킬 수 있고, LRU 알고리즘에 의해서 삭제 되거나 unpersist을 통해서 메모리에서 삭제할 수 있다. cache()는  persist() 에서 저장 옵션을 MEMORY_ONLY로 한 옵션과 동일하다.

데이터 양이 많을 경우에는 DISK에 저장하는 옵션 보다는 차라리 persist하지 않고(Serialize-Deserialize의 오버헤드가 큼), 필요할 때마다 재계산하는 것이 더 빠를 수 있다.
**lazy-execution의 장점은 늦게 실행되는 동안 DAG를 통해서 transformation의 연산의 코스트를 미리 계산해 최적으로 계산한다.**

**Narrow Dependency**
Narrow는 한 노드에서 처리 할 수 있는 일은 모아서


**Wide Dependency**
Wide는 모든 노드에서 작업을 모아서 연산을 수행하기 때문에 Network I/O가 발생하고 느리다.

## 4.Spark On Mesos
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.05.42.png?raw=true)

**Mesos - Master에 Spark 설치**
```
https://spark.apache.org/downloads.html
wget http://d3kbcqa49mib13.cloudfront.net/spark-2.0.1-bin-hadoop2.7.tgz
Spark이 설치된  /home/nkh/spark-2.0.0-bin-hadoop2.7/conf 폴더에서 spark-env.sh.template -> spark-env.sh로 복사

복사된 spark-env.sh파일에 아래 두 개를  설정
Spark에서 Mesos를 사용할  수 있도록
export MESOS_NATIVE_JAVA_LIBRARY=/usr/lib/libmesos.so

Mesos Master 서버에서  python -m SimpleHTTPServer 9914 실행
Spark을  다운 받아 Mesos Slave에서 실행 할 수 있도록 설정
export SPARK_EXECUTOR_URI=http://130.211.188.2:9914/spark-2.0.0-bin-hadoop2.7.tgz

```
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-02%20%EC%98%A4%EC%A0%84%2012.09.25.png?raw=true)

**Spark실행**
```
Client Mode
~/spark-2.0.1-bin-hadoop2.7/bin
bash pyspark --master mesos://10.128.0.2:5050 <- 내부 주소 말고 외부 주소를 사용하면 제대로 동작하지 않음. 다른 설정이 있는지 확인이 필요.
```
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.05.55.png?raw=true)

```
Spark이실행되면 모든 Mesos-Slave에 Spark을 실행할 수 있도록 준비
```
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.06.02.png?raw=true)

**Spark작업 처리 현황**
http://130.211.188.2:4040/executors/
![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.06.07.png?raw=true)

**cluster-mode**
마스터에서 메소스 디스패쳐 실행
```
~/spark-2.0.1-bin-hadoop2.7/sbin
bash start-mesos-dispatcher.sh --master mesos://10.128.0.2:5050

디스패처를 실행시키면 아래와 같이 spark-submit으로 받을 수 있도록 서버를 연다.
Spark Command: /usr/lib/jvm/java-8-oracle/bin/java -cp starting org.apache.spark.deploy.mesos.MesosClusterDispatcher, logging to /home/mesos-master-1/spark-2.0.1-bin-hadoop2.7/logs/spark-mesos-master-1-org.apache.spark.deploy.mesos.MesosClusterDispatcher-1-mesos-master-1.out
mesos-master-1@mesos-master-1:~/spark-2.0.1-bin-hadoop2.7/sbin$ tail -f /home/mesos-master-1/spark-2.0.1-bin-hadoop2.7/logs/spark-mesos-master-1-org.apache.spark.deploy.mesos.MesosClusterDispatcher-1-mesos-master-1.out
16/10/05 08:00:29 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(mesos-master-1); groups with view permissions: Set(); users  with modify permissions: Set(mesos-master-1); groups with modify permissions: Set()
16/10/05 08:00:29 INFO Utils: Successfully started service on port 8081.
16/10/05 08:00:29 INFO MesosClusterUI: Bound MesosClusterUI to 0.0.0.0, and started at http://10.0.2.15:8081
I1005 08:00:29.956048 15739 sched.cpp:226] Version: 1.0.1
I1005 08:00:29.959381 15736 sched.cpp:330] New master detected at master@192.168.56.101:5050
I1005 08:00:29.959638 15736 sched.cpp:341] No credentials provided. Attempting to register without authentication
I1005 08:00:29.960786 15736 sched.cpp:743] Framework registered with 56f0b494-020b-4ef5-99d6-cd36ed1fdccc-0003
16/10/05 08:00:29 INFO MesosClusterScheduler: Registered as framework ID 56f0b494-020b-4ef5-99d6-cd36ed1fdccc-0003
16/10/05 08:00:29 INFO Utils: Successfully started service on port 7077.
16/10/05 08:00:29 INFO MesosRestServer: Started REST server for submitting applications on port 7077
```


클러스터 모드 실행
```
다른 서버에서 스팍을 설치 후 spark-submit으로 프로그램 제출
/home/nkh/spark-2.0.0-bin-hadoop2.7/bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master mesos://10.128.0.2:7077 \
--deploy-mode cluster \
http://130.211.188.2:9914/cluster-test.py    <- 실행해야할 파일 python -m SimpleHTTPServer 9914 띄워서 사용

위 내용은 GCE에서 실행했던 부분
아래 내용은 virtualbox에서 실행

/home/mesos-master-3/spark-2.0.1-bin-hadoop2.7/bin/spark-submit \
--class org.apache.spark.examples.SparkPi --master mesos://192.168.56.101:7077 --deploy-mode cluster \ http://192.168.56.106:9914/test.py


```

![spark](https://github.com/namgunghyeon/wiki/blob/master/images/spark/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-02%20%EC%98%A4%EC%A0%84%2012.45.42.png?raw=true)

https://github.com/namkunghyeon/python_spark


***중요***
```
--master mesos://192.168.56.101:7077 커넥션이 안되는 경우 netstat -pln로 7077포트가 정상적으로 아이피에 바인딩되어 있는지 확인
tcp6       0      0 192.168.56.101:7077     :::*                    LISTEN      15852/java
아래 처럼 되어 있다면
tcp6       0      0 127.0.1.1:7077     :::*                    LISTEN      15852/java

vi /etc/hosts에서
127.0.1.1      mesos-master-1 해당 부분 삭제

정상적이라면 아래 처럼 기록이 되어 있어야함.
192.168.56.101  mesos-master-1
192.168.56.102  mesos-master-2
192.168.56.103  mesos-master-3

```

**출처 :
http://ourcstory.tistory.com/124
https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-mesos.html
https://ogirardot.wordpress.com/2015/05/29/rdds-are-the-new-bytecode-of-apache-spark/
https://vanwilgenburg.wordpress.com/2015/05/10/how-to-run-a-spark-cluster-on-mesos-on-your-mac/
http://bcho.tistory.com/1024**
