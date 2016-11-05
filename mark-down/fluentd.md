## Fluentd정리
### 1.Fluentd란
C와 Ruby로 만들어진 Log Aggregator 시스템으로 단순함과 안전성을 초점으로 두며 대용량 처리에 기반을 두고 만들어졌다. Fluentd의 가장 큰특징이자 장점은 쉽게 Plugin을 만들 수 있고, 많은 Plugin들이 있다. Ruby로 만들어져 Plugin을 gem으로 배포하기 때문에 쉽게 붙일 수 있으며 성능이 필요한 부분은 C언어로 만들어져 있다. 기본 포멧으로 JSON을 이용해 개발 접근성이 쉽다.

![flunentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.00.38.png?raw=true)


### 2.구조
Fluentd를 설치하면, 서버에서 기동되고 있는 서버(또는 애플리케이션)에서 로그를 수집해 중앙 로그 저장소로 전송하는 방식을 가장 기본적인 구조로 Fluentd가 로그 수집 에이전트 역할만하는 구조이다.

![flunentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.00.44.png?raw=true)

더 나아가 Fluentd에서 수집한 로그를 다른 Flunentd로 보내서 받은 Fluentd가 최종적으로 로그 저장소에 저장한다. 이를 통해서 Throtting을 해서 로르 저장소의 용량에 맞게 트랙픽을 조정할 수 있다.
![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.00.48.png?raw=true)

또 한 로그 종류에 따라서 각각 다른 로그 저장소로 라우팅이 가능하다.

![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.00.51.png?raw=true)

Weighted Load Balancing
가중치를 통해서 서버별로 부하를 다르게 분배 할 수 있다.
![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.01.05.png?raw=true)


At-least-Once Semantics
여러번 메시지를 전달 시도를 통해서 적어도 한번의 메시지 전송 성공을 보장하는 방식이며, 데이터 중복은 허용한다.
![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.01.09.png?raw=true)


***Most Once*** 메시지 중복을 허용하지 않고 한번만 전달, 데이터 유실 발생 가능성

***At Least Once*** 한번의 메시지 전송 성공을 보장하는 방식이며, 데이터 중복은 허용

***Exactly Once*** 정확하게 한번의 메시지를 전달하는 방식, 중복과 유실을 허용하지 않는 전달 방식으로 성능에 미지치는 요소가 큼

Fluentd는 Input, Parse, Engine, Filter, Buffer, Output, Formatter 7개의 컴퓨넌트의 구성이 된다. Engine을 제외한 나머지 6개는 플러그인 형태로 제공되어 사용자가 설정이 가능하다.
Input -> Engine -> Output의 흐름으로 이루어지고, Parser, Buffer, Filter, Formatter등은 설정에 따라서 선택적으로 추가 또는 삭제할 수 있다.

![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.01.14.png?raw=true)

Input
:   로그를 수집하는 플러그인으로, 다양한 로그 소스를 지원하고있다. Http, Tail, TCP등 기본적인 플러그인 외에도, 확장 플러그인을 통해서 다양한 포멧을 읽을 수 있다.

Parser(Option)
:   Fluentd에서 지원하지 않는 데이터의 포멧의 경우 Parse플러인을 선택적으로 사용할 수 있다.

Filter(Option)
:   읽어드린 데이터를 Out으로 보내기전에 처리할 수 있는 기능
  - 필터링
  - 데이터 필드 추가
  - 데이터 필드 삭제 또는 특정필드 마킹

Output
:   필터링된 데이터를 로그 저장소에 데이터를 저장한다.(DB, AWS S3, Hadoop...)

Formatter(option)
:   Formatter를 이용하면 쓰는 데이터의 포멧을 정의할 수 있다.

Buffer(option)
:   Input에서 들어온 데이터를 바로 Output으로 보내서 쓰는것이 아니라 중간에 선택적으로 Buffer에 저장하고 Throtting을 할 수 있다. 버퍼는 File, Memory를 사용할 수 있다.

![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.01.25.png?raw=true)


버퍼에는 로그 데이터를 분리하는 Tag 단위로 Chunk가 생성된다. Chunk는 태그별 큐라고 보면된다. Chunk에 데이터가 쌓여 buffer_chunk_limit 만큼 쌓여 Full이 되너가, 또는 설정 값에 의해서 flush_interval 주기가 되면 로그 저장소로 로그를 쓰이 위해서 Queue에 전달된다.


![fluentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-30%20%EC%98%A4%ED%9B%84%2010.01.32.png?raw=true)

***Time***:로그 데이터 생성 시간

***Record***:로그 데이터를 내용으로 JSON 형태로 정의한다.

***Tag***:각 로그 레코드는 Tag를 통해서 로그의 종류가 정해지는데, 이 Tag에 따라서 로그에 대한 필터링, 라우팅과 같은 플러그인에 적용된다.

### 3.설치 및 테스트
![flunentd](https://github.com/namgunghyeon/wiki/blob/master/images/fluentd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-04%20%EC%98%A4%EC%A0%84%2012.09.17.png?raw=true)

**설치**
Ubuntu 14.04
```
http://docs.fluentd.org/articles/install-by-deb

For Trusty,
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent2.sh | sh

실행
sudo /etc/init.d/td-agent restart

sudo /etc/init.d/td-agent start
sudo /etc/init.d/td-agent stop
sudo /etc/init.d/td-agent restart
sudo /etc/init.d/td-agent status

```

```
fluentd-01@ubuntu:/var/log/td-agent$ td-agent -h
Usage: td-agent [options]
    -s, --setup [DIR=/etc/td-agent]  install sample configuration file to the directory
    -c, --config PATH                config file path (default: /etc/td-agent/td-agent.conf)
        --dry-run                    Check fluentd setup is correct or not
        --show-plugin-config=PLUGIN  Show PLUGIN configuration and exit(ex: input:dummy)
    -p, --plugin DIR                 add plugin directory
    -I PATH                          add library path
    -r NAME                          load library
    -d, --daemon PIDFILE             daemonize fluent process
        --no-supervisor              run without fluent supervisor
        --user USER                  change user
        --group GROUP                change group
    -o, --log PATH                   log file path
    -i CONFIG_STRING,                inline config which is appended to the config file on-fly
        --inline-config
        --emit-error-log-interval SECONDS
                                     suppress interval seconds of emit error logs
        --suppress-repeated-stacktrace [VALUE]
                                     suppress repeated stacktrace
        --without-source             invoke a fluentd without input plugins
        --use-v1-config              Use v1 configuration format (default)
        --use-v0-config              Use v0 configuration format
    -v, --verbose                    increase verbose level (-v: debug, -vv: trace)
    -q, --quiet                      decrease verbose level (-q: warn, -qq: error)
        --suppress-config-dump       suppress config dumping when fluentd starts
    -g, --gemfile GEMFILE            Gemfile path
    -G, --gem-path GEM_INSTALL_PATH  Gemfile install path (default: $(dirname $gemfile)/vendor/bundle)
fluentd-01@ubuntu:/var/log/td-agent$
```
**설정 파일**
설정에 대한 기본 내용이 있다.
```ruby
/etc/td-agent/td-agent.conf
####
## Output descriptions:
##
# Treasure Data (http://www.treasure-data.com/) provides cloud based data
# analytics platform, which easily stores and processes data from td-agent.
# FREE plan is also provided.
# @see http://docs.fluentd.org/articles/http-to-td
#
# This section matches events whose tag is td.DATABASE.TABLE
<match td.*.*>
  @type tdlog
  apikey YOUR_API_KEY

  auto_create_table
  buffer_type file
  buffer_path /var/log/td-agent/buffer/td

  <secondary>
    @type file
    path /var/log/td-agent/failed_records
  </secondary>
</match>

## match tag=debug.** and dump to console
<match debug.**>
  @type stdout
</match>

####
## Source descriptions:
##

## built-in TCP input
## @see http://docs.fluentd.org/articles/in_forward
<source>
  @type forward
</source>

## built-in UNIX socket input
#<source>
#  @type unix
#</source>

# HTTP input
# POST http://localhost:8888/<tag>?json=<json>
# POST http://localhost:8888/td.myapp.login?json={"user"%3A"me"}
# @see http://docs.fluentd.org/articles/in_http
<source>
  @type http
  port 8888
</source>

## live debugging agent
<source>
  @type debug_agent
  bind 127.0.0.1
  port 24230
</source>

####
## Examples:
##

## File input
## read apache logs continuously and tags td.apache.access
#<source>
#  @type tail
#  format apache
#  path /var/log/httpd-access.log
#  tag td.apache.access
#</source>

## File output
## match tag=local.** and write to file
#<match local.**>
#  @type file
#  path /var/log/td-agent/access
#</match>

## Forwarding
## match tag=system.** and forward to another td-agent server
#<match system.**>
#  @type forward
#  host 192.168.0.11
#  # secondary host is optional
#  <secondary>
#    host 192.168.0.12
#  </secondary>
#</match>

## Multiple output
## match tag=td.*.* and output to Treasure Data AND file
#<match td.*.*>
#  @type copy
#  <store>
#    @type tdlog
#    apikey API_KEY
#    auto_create_table
#    buffer_type file
#    buffer_path /var/log/td-agent/buffer/td
#  </store>
#  <store>
#    @type file
#    path /var/log/td-agent/td-%Y-%m-%d/%H.log
#  </store>
#</match>
```



**Input Plugins**
- in_tail
```ruby
<source>
  @type tail
  path /home/fluentd-01/logs/message.log
  pos_file /home/fluentd-01/logs/pos_file.pos <- 현재까지 읽은 포지션 저장
  tag test.message
  format none
</source>

<match **>
  type stdout
</match>


root@ubuntu:/home/fluentd-01/logs# ls
message.log  pos_file.pos
root@ubuntu:/home/fluentd-01/logs# echo "namkunghyeon" >> message.log
root@ubuntu:/home/fluentd-01/logs# echo "namkunghyeon" >> message.log
root@ubuntu:/home/fluentd-01/logs# echo "namkunghyeon" >> message.log
root@ubuntu:/home/fluentd-01/logs# echo "namkunghyeon" >> message.log
root@ubuntu:/home/fluentd-01/logs# echo "namkunghyeon" >> message.log
root@ubuntu:/home/fluentd-01/logs# echo "namkunghyeon" >> message.log

2016-11-04 01:23:53 +0900 [info]: following tail of /home/fluentd-01/logs/message.log
2016-11-04 01:23:53 +0900 test.message: {"message":"namkunghyeon"}
2016-11-04 01:24:13 +0900 test.message: {"message":"namkunghyeon"}
2016-11-04 01:24:15 +0900 test.message: {"message":"namkunghyeon"}
2016-11-04 01:24:15 +0900 test.message: {"message":"namkunghyeon"}
2016-11-04 01:24:16 +0900 test.message: {"message":"namkunghyeon"}

```

- in_http
```ruby
<source>
  @type http
  port 8888
  bind 0.0.0.0
  body_size_limit 32m
  keepalive_timeout 10s
</source>

2016-11-04 01:30:41 +0900 [info]: following tail of /home/fluentd-01/logs/message.log
2016-11-04 01:31:24 +0900 test.tag.here: {"action":"login","user":2}
2016-11-04 01:31:25 +0900 test.tag.here: {"action":"login","user":2}
2016-11-04 01:31:26 +0900 test.tag.here: {"action":"login","user":2}
2016-11-04 01:31:27 +0900 test.tag.here: {"action":"login","user":2}
2016-11-04 01:31:27 +0900 test.tag.here: {"action":"login","user":2}
2016-11-04 01:31:27 +0900 test.tag.here: {"action":"login","user":2}
2016-11-04 01:31:28 +0900 test.tag.here: {"action":"login","user":2}

root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;
root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;
root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;
root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;
root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;
root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;
root@ubuntu:/home/fluentd-01/logs# curl -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.tag.here;

http://localhost:8888/<tag> tag부분을 변경해서 로그를 구분해서 받을 수 있다.
```

- in_exec
```ruby
<source>
  @type exec
  command ps -ef | grep td-agent
  tag test.message
  run_interval 1min
  keys k1
</source>


2016-11-04 01:38:12 +0900 [info]: following tail of /home/fluentd-01/logs/message.log
2016-11-04 01:39:12 +0900 test.message: {"k1":"td-agent  3238     1  0 01:38 ?        00:00:00 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid"}
2016-11-04 01:39:12 +0900 test.message: {"k1":"td-agent  3241  3238  0 01:38 ?        00:00:00 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid"}
2016-11-04 01:39:12 +0900 test.message: {"k1":"fluentd+  3250  1717  0 01:38 pts/1    00:00:00 tail -f /var/log/td-agent/td-agent.log"}
2016-11-04 01:39:12 +0900 test.message: {"k1":"td-agent  3251  3241  0 01:39 ?        00:00:00 sh -c ps -ef | grep td-agent"}
2016-11-04 01:39:12 +0900 test.message: {"k1":"td-agent  3252  3251  0 01:39 ?        00:00:00 ps -ef"}
2016-11-04 01:39:12 +0900 test.message: {"k1":"td-agent  3253  3251  0 01:39 ?        00:00:00 grep td-agent"}

2016-11-04 01:40:13 +0900 test.message: {"k1":"td-agent  3238     1  0 01:38 ?        00:00:00 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid"}
2016-11-04 01:40:13 +0900 test.message: {"k1":"td-agent  3241  3238  0 01:38 ?        00:00:00 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid"}
2016-11-04 01:40:13 +0900 test.message: {"k1":"fluentd+  3250  1717  0 01:38 pts/1    00:00:00 tail -f /var/log/td-agent/td-agent.log"}
2016-11-04 01:40:13 +0900 test.message: {"k1":"td-agent  3254  3241  0 01:40 ?        00:00:00 sh -c ps -ef | grep td-agent"}
2016-11-04 01:40:13 +0900 test.message: {"k1":"td-agent  3255  3254  0 01:40 ?        00:00:00 ps -ef"}
2016-11-04 01:40:13 +0900 test.message: {"k1":"td-agent  3256  3254  0 01:40 ?        00:00:00 grep td-agent"}

```

- in_syslog
- in_forward


**Output Plogins**
- out_stdout(Non-Buffered)
- out_file
- **out_forward**


**Formatter**
- csv, tsv, ltsv

**자세한 설정 내용은 아래에서 확인할 수 있다.**
http://docs.fluentd.org/articles/input-plugin-overview

***출처:
http://www.popit.kr/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%8B%A4%EC%8B%9C%EA%B0%84-%EC%B2%98%EB%A6%AC-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC-%EB%B9%84%EA%B5%90%EB%B6%84%EC%84%9D-1/
http://system-server-engineer.tistory.com/32
http://www.slideshare.net/td-nttcom/fluentd-vs-logstash-for-openstack-log-management
http://www.fluentd.org/why
http://bcho.tistory.com/1115
http://blog.seulgi.kim/2014/04/log-aggregator-scribe-flume-fluentd.html
http://docs.fluentd.org/articles/buffer-plugin-overview***
