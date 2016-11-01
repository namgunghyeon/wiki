# Mesos 정리
## 1.Mesos란
데이터센터 내의 자원을 공유/격리를 관리하는 기술 Hadoop, MPI, Hypertable, Spark과 같은 응용 프로그램을 동적 클러스터링 환경에서 리소스 공유와 분리
를 통해 자원 최적화가 가능하게된다. Mesos는 클러스터에서 사용가능한 자원을 추적하고 사용자의 요구에 따라 할당하는 일을한다.
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.55.57.png?raw=true)

분산커널시스템
리눅스 커널과 동일하며 단지 추상화 레벨만 다르다. 모든 머신에서 동작하며, 실행 애플리케이션에 대해 리소스 관리와 스케쥴링 API를 제공한다.
**Mesosphere = Mesos + Marathon + Chronos**
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.01.png?raw=true)


## 2.동작 방식
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.11.png?raw=true)

```
1.슬레이브에 여유 리소스가 생기면, 마스테에게 여유 정보를 통보한다.
2.할당정책에 따라서, 마스터는 얼마나 많은 리소스가 각 프레임워크에 할당되었는지 결정한다.
3.마스터가 제안을 보내고, 스케쥴러는 어느 제안 리소스가 받아 들여졌는지 선택한다.
4.프레임워크가 제안 리소를 받아 들이면, 실행할 작업 내용을 Mesos 에게 보낸다.
5.적합한 리소스를 실행하기위해 할당한 슬레이브에게 마스터는 차례로 작업들을 보낸다.
6.최종적으로 프레임워크는 작업을 실행하게된다.
```

## 3.Mesos 구성방식
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.17.png?raw=true)


## 4.Mesos + Marathon 구성하기
GCE TEST 계정 사용
nkh@encoredtech.com(만료)
설치 방법
http://www.joinc.co.kr/w/man/12/mesos

**마스터 설치**
```
Mesos Master 설치
# add-apt-repository ppa:webupd8team/java
# apt-get update
# apt-get install oracle-java8-installer
# apt-get install oracle-java8-set-default


# apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF
# echo "deb http://repos.mesosphere.com/ubuntu trusty main" >> /etc/apt/sources.list
# apt-get update
# apt-get install -y mesos marathon

# service mesos-slave stop
# echo manual > /etc/init/mesos-slave.override

주키퍼 설정을 한다. 주키퍼 quorum을 구성하는 주키퍼 노드들은 고유의 id를 가지고 있어야 한다. mesos-01은 1, 02는 2, 03은 3을 설정한다.
# cat /etc/zookeeper/conf/myid
1
/etc/mesos-master/quorum
2

/etc/zookeeper/conf/zoo.cfg
server.1=130.211.188.2:2888:3888
server.2=104.154.36.49:2888:3888
server.3=130.211.136.110:2888:3888

/etc/hosts
130.211.188.2   mesos-01
104.154.36.49  mesos-02
130.211.136.110  mesos-03

#echo 192.168.2.1 | sudo tee /etc/mesos-master/ip
sudo cp /etc/mesos-master/ip /etc/mesos-master/hostname

/etc/mesos/zk
zk://130.211.188.2:2181,104.154.36.49:2181,130.211.136.110:2181/mesos

# service zookeeper restart
# service mesos-master restart
# service marathon restart

```

**슬레이브 설치**
```
master 처럼 자바를 먼저 설치 한다.
# apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF
# echo "deb http://repos.mesosphere.com/ubuntu trusty main" >> /etc/apt/sources.list
# apt-get update
# apt-get -y install mesos

# service zookeeper stop
# service mesos-master stop
#echo manual > /etc/init/mesos-master.override

추가된 내용 - IP설정(각 서버별로 모두 설정)
#echo 192.168.2.51(virtualBox) | sudo tee /etc/mesos-slave/ip
#cp /etc/mesos-slave/ip /etc/mesos-slave/hostname

/etc/mesos/zk
zk://130.211.188.2:2181,104.154.36.49:2181,130.211.136.110:2181/mesos

/etc/hosts
master-01: 130.211.188.2
master-02: 104.154.36.49
master-03: 130.211.136.110
slave-01: 104.197.34.5
slave-02: 104.197.66.147
slave-03: 104.199.197.169
slave-04: 104.199.181.246
slave-05: 104.155.237.164

슬래이브 재시작
service mesos-slave restart
```

**도커 사용하기**
```
# echo 'docker,mesos' > /etc/mesos-slave/containerizers
# echo '5mins' > /etc/mesos-slave/executor_registration_timeout
```
**정상적으로 클러스터가 구축되었는지 확인**
```
주키퍼에 클러스터가 정상적으로 접속되었는지 확인
/usr/share/zookeeper/bin/zkCli.sh -server 마스터 IP
[zk: localhost(CONNECTED) 0] ls /
[mesos, zookeeper, marathon]
```
**mesos**
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.29.png?raw=true)

**marathon**
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.34.png?raw=true)


## 5.Mesos위에  Dokcer로 Python 기본 서버 실행하기
**사전 작업**
```
설치한 모든 슬레이브에 도커가 설치되어 있어야 한다.
도커 설치 참고: https://docs.google.com/document/d/14ahGvBQJltmJG2MNNlIlaGeonbWdRocMi3Bg4h8yf3s/edit
도커 설치 이후 (슬래이브에 설정)
# echo 'docker,mesos' > /etc/mesos-slave/containerizers
# echo '5mins' > /etc/mesos-slave/executor_registration_timeout
```
**슬래이브 재시작**
```
service mesos-slave restart
```

**실행**
```
Marathon UI에서 Docker로 Python 기본 서버 실행
https://mesosphere.github.io/marathon/docs/native-docker.html(Bridged Networking Mode 부분 참고)
http://104.197.66.147:31354/ 접속 해당 포트는 8080으로 열었는데 31354 포트로 접속이 가능??(도커 설정하는 부분을 좀 더 확인해봐야함.)
http://130.211.188.2:8080에서 설정한 정보 확인
```


## 6.Python으로 Mesos 프레임워크 만들기
**프레임워크를 만들기 위한 사전 작업**
```
메소스 파이선 모듈을 pip로 설치할 수 없어
아래 링크에서 native, interface, executor egg 파일들 다운 받아  easy_install로 설치 후 메소스 파이선 모듈을 사용한다.
https://github.com/namgunghyeon/mesos-alpine

설치 후 export PYTHONPATH=${PYTHONPATH}:/usr/lib/python2.7/site-packages/ 를 통해서 경로를 잡아준다.
https://github.com/namgunghyeon/mesos_scheduler

```

**프레임워크 예제**
```
파이선 모듈 설치 및 프레임워크 예제
http://www.bench87.com/content/3
```

위 깃허브에 있는 소스를 Mesos 마스터에서 실행시키면 아래와 같이 실행된다.

![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.41.png?raw=true)


```
메소스 관리 화면에서 위 같이 프로세스가 실행됨을 볼 수 있다.  깃허브 소스에서 user는 슬레이브에 존재하는 user이여야 한다.
동일한 user로 여러개를 실행 시킬 경우 먼저 실행된 user가 Cpu, Mem을 선점하고 있어 실행 시킬 수 없다. 먼저 선점한 프로세스가 죽거나 처리가 완료되면 처리가 가능하다.

깃허브  소스에서 new_task에서 CPU와 메모리를 1.0 전체 사용하고 있어 여러개의 프로세스를 실행 시킬 수 없지만 아래 와 같이 0.3만 사용하도록 변경하면
CPU, Mem을 할당 가능할 때 까지 할당하면서 여러개의 프로세스를 실행 시킬 수 있다.

```

![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.48.png?raw=true)
![mesos](https://github.com/namgunghyeon/wiki/blob/master/images/mesos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.56.53.png?raw=true)

적절하게 value 값을 변경해 Cluster로 처리가 가능하다.

**출처:
http://mesos.apache.org/
http://www.yongbok.net/blog/apache-mesos-cluster-resource-management/
http://www.slideshare.net/songaal/paa-s-mesosmesosphere
https://mesosphere.github.io
http://blog.iamartin.com/vmware-mesos-install/
http://qiita.com/TsuyoshiUshio@github/items/9d8f5b952b635d94ae6f
https://www.digitalocean.com/community/tutorials/how-to-configure-a-production-ready-mesosphere-cluster-on-ubuntu-14-04
http://www.bench87.com/content/3**
