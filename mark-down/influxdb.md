# InfluxDB정리
## 1.InfluxDB란
Time-series DB는 데이터베이스 중에서도 시계열 데이터를 저장하고 활용하는데 특화된 형태의 데이터베이스이다.
- InfluxDB는 과거에 data store를 위해 구글이 만든 key/value database libary를 사용했었고 현재는 자체적으로 만든 TSM Engine으로 변경했다.
  - leveldb - a fast and lightweight key/value DB libary
  - TSM(Time Structured Merge) Engine https://www.influxdata.com/new-storage-engine-time-structured-merge-tree
- SQL-like Query Language를 지원한다. group by, join 복수개의 time series를 Merge도 가능한다.
- Cluster에 새로운 Node만 추가하면 쉽게 scale-out할 수 있다.**(현재는 정책이 변경되어 enterprise에서만 정식으로 HA를 지원하고 오픈 소스로 사용할 때는 Relay를 구성해서 사용가능하다.)**
- 지정된 시간마다 Query를 실행하고 실행된 결과를 저장할 수 있다.(crontab 느낌)
- HA는 relay https://docs.influxdata.com/influxdb/v1.1/high_availability/relay/ 를 통해 가능하다.
- Go언어로 만들어져 있다.

## 2. 설치
-standalone
https://docs.influxdata.com/influxdb/v1.1/introduction/installation/

레파지토리에 추가
```
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

influxdb설치
```
sudo apt-get update && sudo apt-get install influxdb
sudo service influxdb start
```

configuration
```
/etc/influxdb/influxdb.conf
```

admin page
```
1.1 버전부터 admin page는 deprecated
Note: The Admin UI is deprecated as of InfluxDB 1.1 and will be removed entirely in a subsequent version.
We recommend using Chrongraf or Grafana as a replacement.
```

cli
```
옵션 정보
https://docs.influxdata.com/influxdb/v1.1/tools/shell/

ubuntu@ubuntu:~$ influx
Visit https://enterprise.influxdata.com to register for updates, InfluxDB server management, and monitoring.
Connected to http://localhost:8086 version 1.1.0
InfluxDB shell version: 1.1.0

> show databases;
name: databases
name
----
_internal

> create database test
> show databases;
name: databases
name
----
_internal
test
> use test

```

## 3.query
https://docs.influxdata.com/influxdb/v1.1/query_language/spec/#identifiers


## 4.Realy(HA구성)
Enterprise를 사용할 경우 HA를 다른 설정없이 지원해준다.
![influxdb](https://raw.githubusercontent.com/namgunghyeon/wiki/9b03177d51f0f1d64bd96e34848d618a429b11f2/images/infulxdb/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-20%20%EC%98%A4%EC%A0%84%202.14.40.png)

https://github.com/influxdata/influxdb-relay/blob/master/README.md

 **Go 설치**
참고
https://www.digitalocean.com/community/tutorials/how-to-install-go-1-6-on-ubuntu-14-04

```
sudo mv go /usr/local
sudo vi ~/.profile
export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=/home/ubuntu/go
```

**influxdb-relay 다운로드 및 설치**
https://golang.org/dl/
sudo curl -O https://storage.googleapis.com/golang/go1.7.3.linux-amd64.tar.gz


```
$ # Install influxdb-relay to your $GOPATH/bin
$ go get -u github.com/influxdata/influxdb-relay
$ # Edit your configuration file
$ cp $GOPATH/src/github.com/influxdata/influxdb-relay/sample.toml ./relay.toml
$ vim relay.toml
$ # Start relay!
$ $GOPATH/bin/influxdb-relay -config relay.toml
```

**Configuration**
```
[[http]]
# Name of the HTTP server, used for display purposes only.
name = "example-http"

# TCP address to bind to, for HTTP server.
bind-addr = "127.0.0.1:9096"

# Enable HTTPS requests.
ssl-combined-pem = "/etc/ssl/influxdb-relay.pem"

# Array of InfluxDB instances to use as backends for Relay.
output = [
    # name: name of the backend, used for display purposes only.
    # location: full URL of the /write endpoint of the backend
    # timeout: Go-parseable time duration. Fail writes if incomplete in this time.
    # skip-tls-verification: skip verification for HTTPS location. WARNING: it's insecure. Don't use in production.
    { name="local1", location="http://127.0.0.1:8086/write", timeout="10s" },
    { name="local2", location="http://127.0.0.1:7086/write", timeout="10s" },
]

[[udp]]
# Name of the UDP server, used for display purposes only.
name = "example-udp"

# UDP address to bind to.
bind-addr = "127.0.0.1:9096"

# Socket buffer size for incoming connections.
read-buffer = 0 # default

# Precision to use for timestamps
precision = "n" # Can be n, u, ms, s, m, h

# Array of InfluxDB instances to use as backends for Relay.
output = [
    # name: name of the backend, used for display purposes only.
    # location: host and port of backend.
    # mtu: maximum output payload size
    { name="local1", location="127.0.0.1:8089", mtu=512 },
    { name="local2", location="127.0.0.1:7089", mtu=1024 },
]
```

## 5.Kapacitor


작업 중

출처 :
http://www.cubrid.org/wiki_tools/entry/cubrid-monitoring-dashboard
http://www.popit.kr/influxdb_telegraf_grafana_1/
http://www.popit.kr/influxdb_telegraf_grafana_2/
http://www.popit.kr/influxdb_telegraf_grafana_3/
