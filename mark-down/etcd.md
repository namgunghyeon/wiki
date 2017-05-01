## etcd정리
### 1.etcd란
CoreOS의 핵심 컴포넌트중 하나로 키 체인을 관리하는 etcd서비스이다.
CoreOS는 도커 컨테이너 역활로 활용되며 일반적인 OS의 apt-get, yum기능을 빼고 필요한 핵심 기능만 있는 OS

etcd는 키체인을 저장하는 저장소와 같은 것으로 여러대의 CoreOS가 클러스터로 구성되어 있는 상황에서 하나의OS 인스턴스를 리더로해, 한곳에서 키값이 변경될 경우 약 1초에 1000개의 키를 동기화 할 수 있도록 빠른 서비를 제공한다. (Ubunut에 설치도 가능한다.)

![etcd](https://raw.githubusercontent.com/namgunghyeon/wiki/1280f69ffebb9ddb9449195270c093de01f203b4/images/etcd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-12-11%20%EC%98%A4%EC%A0%84%2011.55.43.png)

Go언어와 [Raft](https://raft.github.io/)프레임워크를 이용해 작성되어 있다.
사용 방법은 [API 가이드](https://github.com/coreos/etcd/blob/v3.0.15/Documentation/v2/api.md)에 자세하게 나와있다.



Install etcd 2.0
```
curl -L  https://github.com/coreos/etcd/releases/download/v2.0.0/etcd-v2.0.0-linux-amd64.tar.gz -o etcd-v2.0.0-linux-amd64.tar.gz
tar xzvf etcd-v2.0.0-linux-amd64.tar.gz
cd etcd-v2.0.0-linux-amd64
mkdir /opt/bin
cp etcd* /opt/bin
```

여러가지 기능 중 Watch를 사용해보기

NodeJS Watch Example
etcd가 설치된 곳에서
```
curl http://192.168.56.210:2379/v2/keys/message -XPUT -d value="Hello etcd"
```
NodeJs에서  `message`키로 watch를 하고 있어 이벤트 발생
[Node-example](https://github.com/namgunghyeon/etcd_example)


Python Watch Example


https://coreos.com/etcd/
https://github.com/coreos/etcd/blob/v3.0.15/Documentation/v2/api.md
