# Chronos 정리
## 1.Chronos란

Chronos는 Unix기반 운영체제에서 실시간 기반의 ***스케쥴러인 역활을하는 Mesos 프레임워크이다.*** 주기적으로 실행하는 작업이라면 Chronos를 이용한다.
Chronos는 실행 결과를 쉽게 확인할 수 있고, Mesos 클러스터를 통해 작업을 실행하기 때문에 서버 상태와 상관 없이 항상 실행이 보장된다.
Docker를 지원한다. Airbnb에서는 Chronos라는 잡스케쥴러 관리 시스템에서 잡 자원 할당에 이 Mesos 메소스 프레임워크를 사용하고 있다.(2014)


![Chronos 구조](https://github.com/namgunghyeon/wiki/blob/master/images/chronos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.06.58.png?raw=true)

## 2.Chronos 설치
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF
DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')
CODENAME=$(lsb_release -cs)

# Add the repository
echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" | \ sudo tee/etc/apt/sources.list.d/mesosphere.list
sudo apt-get -y update


	위 과정은 메소스를 설치해서 진행하지 않아도 OK
sudo apt-get -y install chronos
sudo service chronos start
tail -f /var/log/syslog
```

### 접속 주소 http://130.211.188.2:4400/#


## 3.Python Batch를 Docker로 Chronos에서 사용하기
```

1. Python으로 만든 프로젝트를 도커로 빌드한다.
2. 빌드된 도커 파일을 Docker Hub에 올린다.
3. 스케쥴 JSON파일을 생성한다.
4. 생성된 스케쥴 파일을 Chronos API로 전달한다.
    curl -L -H "Content-Type: application/json" -X POST -d@schedule.json http://130.211.188.2:4400/scheduler/iso8601
5.Chronos UI화면에 추가되었는지 확인한다.
```

![chrono 실행 이미지](https://github.com/namgunghyeon/wiki/blob/master/images/chronos/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.07.04.png?raw=true)

참고 : https://gist.github.com/rji/5e18866f3f45ad3b8702
