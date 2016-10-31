# Dokcer정리
## 1.Docker란
LCX(LinuX Containers)로 만든 컨테이너는 고유의 파일 시스템, 프로세스, 네트워크 공간을 가진다. 가상 머신 처럼 독립적이고 격리된 공간.
컨테이너는 호스트의 커널을 공유하는 공간. 컨테이너 안에는 따로 커널이나 각종 드라이버를 따로 넣을 필요가 없다. 이 때문에 가상머신이 아닌 컨테이너로 불린다.

Docker는 가상화 프레임워크, 컨테이너 기반 가상화를 사용한다. 일반적인 가상화 방식은 호스트OS와 게스트 OS가 따로 있고, 하이퍼바이저를 통해 그 위에 수십개의 게스트 OS를 돌린다. 게스트 OS가 독자적으로 돌아가는 가상 컴퓨터이다.
Docker는 게스트 OS를 두지 않고 호스트 OS를 사용한다. 호스트 OS의 시스템 콜을 직접 호출해 매우 빠른 속도로 동작할 수 있다.

***컨테이너 기반 가상화는 호스트 OS를 그대로 공유하고 유저 스페이스에서 가상화를 제공한다.***

![도커](https://github.com/namgunghyeon/wiki/blob/master/images/docker/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.59.31.png?raw=true)

## 2.이미지와 컨테이너
도커에서 중요한 개념 두가지 이미지와 컨테이너
이미지는 서비스에 필요한 프로그램, 라이브러리, 소스 코드 등을 묶은 파일을 뜻한다. 운영체제로 생각하면 이미지는 실행파일이고 컨테이너는 프로세스라고 할 수 있다. 이미지는 Ubuntu, CentOS등의 리눅스 배포판 이미지를 기반으로 만든다.

![도커 이미지 컨테이너](https://github.com/namgunghyeon/wiki/blob/master/images/docker/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.59.36.png?raw=true)

## 3.Docker VS Virtual
리눅스 컨테이너 기술은 가상화와 비슷한 기술
가상화는 반드시 하이퍼바이저라는 기술이 있어야한다. 하이퍼바이저는 하나의 컴퓨터에서 어려개의 운영체제를 사용할 수 있게 도와주는 기술이다. 가상화 환경은 가장 밑단에 깔린 호스트 OS를 공유한다. 도커는 게스트 OS를 두지 않고 호스트 OS 커널을 바로 사용한다. 하이퍼바이저 대신 도커 엔진이 올라가 호스트 OS와 여러개 애플리케이션을 연결해주는 역활을 한다. 도커를 사용하면 가상화보다는 내부에서 더 적은 일을 처리하고, 애플리케이션을 좀 더 빠르게 효율적으로 실행시킬 수 있다.

![도커 가상화](https://github.com/namgunghyeon/wiki/blob/master/images/docker/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%209.59.41.png?raw=true)


## 4.Docker 설치 및 실행
GCE TEST 계정에 설치

### 설치
```
wget -qO- https://get.docker.com/ | sh <- 자동으로 알아서 설치해준다.
```

```
이미지 검색  
docker search ubuntu

최신 우분투 이미지 받기
docker pull ubuntu:latest

모든 이지미 출력
docker images

컨테이너를 만들고 bash shell실행
docker run -i -t --name hello ubuntu /bin/bash
-i interactive
-t pseudo-tty
--name 컨테이너 이름 설정

모든 컨테이너 출력
docker ps -a

이전에 만들었던 컨테이너 접속
docker start -i hello
docker attach  hello

컨테이너 외부에서 내부 명령어 실행
docker exec hello echo "Hello Word"

컨테이너 중지
docker stop hello

컨테이너 삭제
docker rm hello


이미지 삭제
docker rmi ubuntu:latest

```

#### Docker 이미지 만들기

```
FROM ubuntu:14.04
MAINTAINER Foo Bar foo@bar.com

RUN apt-get update
RUN apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
RUN chown -R www-data:www-data /var/lib/nginx

VOLUME ["/data", "/etc/nginx/site-enabled", "/var/log/nginx"]

WORKDIR /etc/nginx


CMD ["nginx"]


EXPOSE 80
EXPOSE 443
```

```
생성
docker build --tag hello:0.1 .
실생하기
docker run --name hello-nginx -d -p 80:80 -v /root/data:/data hello:0.1

-d 백그라운드
-p 호스트 80과 컨테이터 80 포트를 연결
-v /root/data:/data 옵션으로 /root/data디렉토리를 data 디렉토리에 연결

```

## 5.Docker Hub 사용하기(Git Hub와 연결 가능)
https://hub.docker.com/
https://docs.docker.com/engine/tutorials/dockerrepos/

```
docker build -t namkunghyeon/hello-nginx .
docker push namkunghyeon/hello-nginx
docker run --name hello-nginx namkunghyeon/hello-nginx

docker login
접속 주소
http://107.167.180.77:80
```


### 6. TODO
도커를 실제로 사용하지 않으면 기본 명령어와 숙지할 부분이 많아 계속 사용해봐야 함


***출처:
http://documents.docker.co.kr/
http://containertutorials.com/docker-compose/flask-simple-app.html***
