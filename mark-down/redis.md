# Redis정리
## 1.Redis란
Remote Dictionary server
휘발성이면서 영속성을 가진 key-value 형 스토어로 String, Hash, List, Sets, Sotred Set, Btimap, Hyperloglogs등 다양한 데이터 구조를 저장할 수 있어, data structure server라고 부르기도 한다.

메모리에 데이터를 쓰는 In Memoery 데이터베이스 및 NoSQL로 분류되며, 메모리에 데이터를 저장하고 있어 빠른 읽기 쓰기 서비를 제공한다.

## 2.설치
레디스 다운 및 설치
```
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install tcl8.5
sudo apt-get install make

wget http://download.redis.io/releases/redis-3.2.5.tar.gz
tar -xvf redis-3.2.5.tar.gz
cd redis-3.2.5

make
```
서버 실행
```
sudo ./redis-server /home/ubuntu/redis-3.2.5/redis.conf
```

클라이언트 접속
```
sudo ./redis-cli -h 127.0.0.1
```

## 3.자료구조
##### String
```
SET : 저장
GET : 가져오기

127.0.0.1:6379> set testKey testValue
OK
127.0.0.1:6379> get testKey
"testValue"

INCR: 증가
DECR: 감소

127.0.0.1:6379> set counter 1
OK
127.0.0.1:6379> INCR counter
(integer) 2
127.0.0.1:6379> get counter
"2"
127.0.0.1:6379> INCR counter
(integer) 3
127.0.0.1:6379> get counter
"3"
127.0.0.1:6379> DECR counter
(integer) 2
127.0.0.1:6379> get counter
"2"

MSET : 여러개 저장
MGET : 여러개 가져오기
127.0.0.1:6379> mset key1 test1 key2 test2
OK
127.0.0.1:6379> mget key1 key2
1) "test1"
2) "test2"

```

##### List
```
RPUSH : 맨 뒤에서 넣기
LPUSH : 맨 앞쪽(왼쪽)에서 넣기

LRANGE : 범위로 데이터 읽어 오기, 시작값 끝 값
LRANGE testList 0 -1 (0번째 부터 마지막까지)
-1은 오른쪽 인덱스 끝을 의미

127.0.0.1:6379> rpush testList A
(integer) 1
127.0.0.1:6379> rpush testList B
(integer) 2
127.0.0.1:6379> lrange testList 0 -1
1) "A"
2) "B"


127.0.0.1:6379> lpush testList C
(integer) 3
127.0.0.1:6379> lpush testList D
(integer) 4
127.0.0.1:6379> lpush testList E
(integer) 5
127.0.0.1:6379> lrange testList 0 -1
1) "E"
2) "D"
3) "C"
4) "A"
5) "B"

POP : 꺼내서 읽기 (리스트에서 데이터가 삭제)

127.0.0.1:6379> rpop testList
"B"
127.0.0.1:6379> rpop testList
"A"
127.0.0.1:6379> rpop testList
"C"
127.0.0.1:6379> rpop testList
"D"
127.0.0.1:6379> rpop testList
"E"
127.0.0.1:6379> rpop testList
(nil)

LTRIM : 특정 영역만 남기고 삭제, 시작값 끝 값

127.0.0.1:6379> rpush testList 1 2 3 4 5
(integer) 5
127.0.0.1:6379> lrange testList 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
127.0.0.1:6379> ltrim testList 0 1
OK
127.0.0.1:6379> lrange testList 0 -1
1) "1"
2) "2"
```

##### Hash
```
file와 value의 쌍으로 이루어진 테이블을 저장할 수 있는 테이터 구조

127.0.0.1:6379>  hmset user:1000 username yundream birthyear 1997 verified 1
OK
127.0.0.1:6379> hget user:1000 username
"yundream"
127.0.0.1:6379> hget user:1000 birthyear
"1997"
127.0.0.1:6379> hget user:1000 1997
(nil)
127.0.0.1:6379> hget user:1000 verified
"1"
127.0.0.1:6379> hgetall user:1000
```
##### Set
```
정렬되지 않은 String집합

sadd myset 1 2 3
127.0.0.1:6379> sadd myset 1 2 3
(integer) 3
127.0.0.1:6379> smembers myset
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> SISMEMBER myset 5
(integer) 0
127.0.0.1:6379> SISMEMBER myset 3
(integer) 1
```

##### Sorted Set
```
127.0.0.1:6379> zadd hackers 1940 "Alan Kay"
(integer) 1
127.0.0.1:6379> zadd hackers 1957 "Sophie Wilson"
(integer) 1
127.0.0.1:6379> zadd hackers 1953 "Richard Stallman"
(integer) 1
127.0.0.1:6379> zadd hackers 1949 "Anita Borg"
(integer) 1
127.0.0.1:6379> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
127.0.0.1:6379> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
127.0.0.1:6379> zadd hackers 1916 "Claude Shannon"
(integer) 1
127.0.0.1:6379> zadd hackers 1969 "Linus Torvalds"
(integer) 1
127.0.0.1:6379> zadd hackers 1912 "Alan Turing"
(integer) 1

127.0.0.1:6379> ZRANGE hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"

127.0.0.1:6379> ZREVRANGE hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"

127.0.0.1:6379> ZREVRANGE hackers 0 -1 withscores
 1) "Linus Torvalds"
 2) "1969"
 3) "Yukihiro Matsumoto"
 4) "1965"
 5) "Sophie Wilson"
 6) "1957"
 7) "Richard Stallman"
 8) "1953"
 9) "Anita Borg"
10) "1949"
11) "Alan Kay"
12) "1940"
13) "Claude Shannon"
14) "1916"
15) "Hedy Lamarr"
16) "1914"
17) "Alan Turing"
18) "1912"

127.0.0.1:6379> ZRANGEBYSCORE hackers -inf 1950 withscores
 1) "Alan Turing"
 2) "1912"
 3) "Hedy Lamarr"
 4) "1914"
 5) "Claude Shannon"
 6) "1916"
 7) "Alan Kay"
 8) "1940"
 9) "Anita Borg"
10) "1949"
127.0.0.1:6379> ZREMRANGEBYSCORE hackers 1940 1960
(integer) 4

Lexicographical scores
score값이 같을 경우 값을 비교해서 정렬한다. memcmp함수를 사용한다.

127.0.0.1:6379> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0 "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon" 0 "Linus Torvalds" 0 "Alan Turing"
(integer) 4
127.0.0.1:6379> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
```

##### Bitmaps
```
0과 1을 가지는 상태를 나타낼 수 있다.
127.0.0.1:6379> setbit visitor 10 1
(integer) 0
127.0.0.1:6379> getbit visitor 10
(integer) 1
127.0.0.1:6379> getbit visitor 11
(integer) 0
```

##### HyperLogLogs
```
데이터셋에서 집합에서 유일한 원소의 갯수를 검사하기 위해서 사용되는 알고리즘
127.0.0.1:6379> PFADD hll foo bar zap
(integer) 1
127.0.0.1:6379> PFADD hll zap zap zap
(integer) 0
127.0.0.1:6379> PFADD hll foo bar
(integer) 0
127.0.0.1:6379> PFCOUNT hll
(integer) 3
```
## 4.Replication
master : read/write
slave : read

master : 192.168.56.203(7000)
slave : 192.168.56.204(7001), 192.168.56.205(7002), 192.168.56.206(7003)

```
설정
/home/ubuntu/redis-3.2.5/src

utils폴더에 있는 install_server 파일을 통해서 config 파일 생성
ubuntu@ubuntu:~/redis-3.2.5/utils$  sudo ./install_server.sh
[sudo] password for ubuntu:
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 7000(master), 7001(slave), 7002(slave), 7003(slave)
Please select the redis config file name [/etc/redis/7003.conf]
Selected default - /etc/redis/7003.conf
Please select the redis log file name [/var/log/redis_7003.log]
Selected default - /var/log/redis_7003.log
Please select the data directory for this instance [/var/lib/redis/7003]
Selected default - /var/lib/redis/7003
Please select the redis executable path [] /home/ubuntu/redis-3.2.5/src
Selected config:
Port           : 7003
Config file    : /etc/redis/7003.conf
Log file       : /var/log/redis_7003.log
Data dir       : /var/lib/redis/7003
Executable     : /home/ubuntu/redis-3.2.5/src
Cli Executable : /home/ubuntu/redis-3.2.5/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/7003.conf => /etc/init.d/redis_7003
Installing service...

sudo vi /etc/redis/7000.conf <- master
bind 192.168.56.203
masterauth testserver
requirepass testserver

sudo vi /etc/redis/7000.conf <- slave
bind 192.168.56.204
slaveof 192.168.56.203 7000
masterauth testserver
requirepass testserver
repl-ping-slave-period 10
repl-timeout 60

나머지 slave파일도 bind만 빼고 동일하게 수정



sudo ./redis-server /etc/redis/7000.conf
sudo ./redis-server /etc/redis/7001.conf
sudo ./redis-server /etc/redis/7002.conf
sudo ./redis-server /etc/redis/7003.conf

10995:M 12 Nov 20:19:44.963 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10995:M 12 Nov 20:19:44.963 # Server started, Redis version 3.2.5
10995:M 12 Nov 20:19:44.963 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
10995:M 12 Nov 20:19:44.963 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
10995:M 12 Nov 20:19:44.963 * The server is now ready to accept connections on port 7000
10995:M 12 Nov 20:20:50.998 * Slave 192.168.56.204:7001 asks for synchronization
10995:M 12 Nov 20:20:50.999 * Full resync requested by slave 192.168.56.204:7001
10995:M 12 Nov 20:20:50.999 * Starting BGSAVE for SYNC with target: disk
10995:M 12 Nov 20:20:51.000 * Background saving started by pid 11008
11008:C 12 Nov 20:20:51.005 * DB saved on disk
11008:C 12 Nov 20:20:51.006 * RDB: 6 MB of memory used by copy-on-write
10995:M 12 Nov 20:20:51.075 * Background saving terminated with success
10995:M 12 Nov 20:20:51.076 * Synchronization with slave 192.168.56.204:7001 succeeded
10995:M 12 Nov 20:20:53.279 * Slave 192.168.56.205:7002 asks for synchronization
10995:M 12 Nov 20:20:53.279 * Full resync requested by slave 192.168.56.205:7002
10995:M 12 Nov 20:20:53.280 * Starting BGSAVE for SYNC with target: disk
10995:M 12 Nov 20:20:53.280 * Background saving started by pid 11009
11009:C 12 Nov 20:20:53.286 * DB saved on disk
11009:C 12 Nov 20:20:53.287 * RDB: 6 MB of memory used by copy-on-write
10995:M 12 Nov 20:20:53.386 * Background saving terminated with success
10995:M 12 Nov 20:20:53.386 * Synchronization with slave 192.168.56.205:7002 succeeded
10995:M 12 Nov 20:20:54.899 * Slave 192.168.56.206:7003 asks for synchronization
10995:M 12 Nov 20:20:54.900 * Full resync requested by slave 192.168.56.206:7003
10995:M 12 Nov 20:20:54.900 * Starting BGSAVE for SYNC with target: disk
10995:M 12 Nov 20:20:54.901 * Background saving started by pid 11010
11010:C 12 Nov 20:20:54.908 * DB saved on disk
11010:C 12 Nov 20:20:54.909 * RDB: 6 MB of memory used by copy-on-write
10995:M 12 Nov 20:20:54.995 * Background saving terminated with success
10995:M 12 Nov 20:20:54.995 * Synchronization with slave 192.168.56.206:7003 succeeded

접속
ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.203 -p 7000 -a testserver
192.168.56.203:7000> keys *
(empty list or set)
192.168.56.203:7000> set testkey testvalue
OK
192.168.56.203:7000>

slave에서 확인
ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> keys *
1) "testkey"

```


## 5.Sentinel

master : 192.168.56.203(7000)
slave : 192.168.56.204(7001), 192.168.56.205(7002), 192.168.56.206(7003)
sentinel : 192.168.56.204(8000), 192.168.56.205(8001), 192.168.56.206(8002)
```
설정
/home/ubuntu/redis-3.2.5/src
sential : 192.168.56.204(8000), 192.168.56.205(8001), 192.168.56.206(8002)
sudo cp sentinel.conf /etc/redis/sentinel8000.conf
sudo cp sentinel.conf /etc/redis/sentinel8001.conf
sudo cp sentinel.conf /etc/redis/sentinel8002.conf

vi /etc/redis/sentinel8000.conf
port 8000
bind 192.168.56.204
sentinel myid 92e14e2c3180c1571f3bd1cca3c561067cf15ce5
sentinel monitor stn-master 192.168.56.206 7003 2
sentinel down-after-milliseconds stn-master 2000
sentinel failover-timeout stn-master 90000
sentinel auth-pass stn-master testserver

위와 같이 설정 후 모든 sentinel 서버에 복사 포트 번호와 아이피를 각 서버에 맞게 수정


실행
sudo ./redis-sentinel /etc/redis/sentinel8000.conf
sudo ./redis-sentinel /etc/redis/sentinel8001.conf
sudo ./redis-sentinel /etc/redis/sentinel8002.conf



1846:X 13 Nov 20:22:06.613 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 8000
 |    `-._   `._    /     _.-'    |     PID: 1846
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

1846:X 13 Nov 20:22:06.618 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1846:X 13 Nov 20:22:06.618 # Sentinel ID is 92e14e2c3180c1571f3bd1cca3c561067cf15ce5
1846:X 13 Nov 20:22:06.619 # +monitor master stn-master 192.168.56.206 7003 quorum 2
1846:X 13 Nov 20:22:06.620 * +slave slave 192.168.56.205:7002 192.168.56.205 7002 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:22:06.626 * +slave slave 192.168.56.204:7001 192.168.56.204 7001 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:22:06.632 * +slave slave 192.168.56.203:7000 192.168.56.203 7000 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:22:16.013 * +sentinel sentinel a67ff97d345c12c2d63270a5e6a6e66da69138e2 192.168.56.205 8001 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:22:22.682 * +sentinel sentinel d123a6538a15c9c162d2d01242d109e9bc8b10b6 192.168.56.206 8002 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.178 # +sdown master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.237 # +odown master stn-master 192.168.56.206 7003 #quorum 3/2
1846:X 13 Nov 20:23:03.238 # +new-epoch 1
1846:X 13 Nov 20:23:03.238 # +try-failover master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.246 # +vote-for-leader 92e14e2c3180c1571f3bd1cca3c561067cf15ce5 1
1846:X 13 Nov 20:23:03.250 # a67ff97d345c12c2d63270a5e6a6e66da69138e2 voted for a67ff97d345c12c2d63270a5e6a6e66da69138e2 1
1846:X 13 Nov 20:23:03.260 # d123a6538a15c9c162d2d01242d109e9bc8b10b6 voted for 92e14e2c3180c1571f3bd1cca3c561067cf15ce5 1


ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.203 -p 7000 -a testserver
192.168.56.203:7000> info replication
# Replication
role:master
connected_slaves:3
slave0:ip=192.168.56.204,port=7001,state=online,offset=75754,lag=0
slave1:ip=192.168.56.205,port=7002,state=online,offset=75754,lag=1
slave2:ip=192.168.56.206,port=7003,state=online,offset=75754,lag=0
master_repl_offset:75910
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:75909

테스트
ubuntu@ubuntu:~$ ps -ef | grep redis
root     10330     1  0 20:26 ?        00:00:00 ./redis-server 192.168.56.206:7003
root     10341 10044  0 20:28 pts/2    00:00:00 sudo ./redis-sentinel /etc/redis/sentinel8002test.conf
root     10342 10341  0 20:28 pts/2    00:00:00 ./redis-sentinel 192.168.56.206:8002 [sentinel]
ubuntu   10451 10436  0 20:29 pts/0    00:00:00 grep --color=auto redis

11846:X 13 Nov 20:23:03.322 # +elected-leader master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.323 # +failover-state-select-slave master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.406 # +selected-slave slave 192.168.56.204:7001 192.168.56.204 7001 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.406 * +failover-state-send-slaveof-noone slave 192.168.56.204:7001 192.168.56.204 7001 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:03.465 * +failover-state-wait-promotion slave 192.168.56.204:7001 192.168.56.204 7001 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:04.295 # +promoted-slave slave 192.168.56.204:7001 192.168.56.204 7001 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:04.295 # +failover-state-reconf-slaves master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:04.343 * +slave-reconf-sent slave 192.168.56.203:7000 192.168.56.203 7000 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:05.317 * +slave-reconf-inprog slave 192.168.56.203:7000 192.168.56.203 7000 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:05.317 * +slave-reconf-done slave 192.168.56.203:7000 192.168.56.203 7000 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:05.406 # -odown master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:05.406 * +slave-reconf-sent slave 192.168.56.205:7002 192.168.56.205 7002 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:06.382 * +slave-reconf-inprog slave 192.168.56.205:7002 192.168.56.205 7002 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:07.414 * +slave-reconf-done slave 192.168.56.205:7002 192.168.56.205 7002 @ stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:07.489 # +failover-end master stn-master 192.168.56.206 7003
1846:X 13 Nov 20:23:07.489 # +switch-master stn-master 192.168.56.206 7003 192.168.56.204 7001
1846:X 13 Nov 20:23:07.490 * +slave slave 192.168.56.203:7000 192.168.56.203 7000 @ stn-master 192.168.56.204 7001
1846:X 13 Nov 20:23:07.490 * +slave slave 192.168.56.205:7002 192.168.56.205 7002 @ stn-master 192.168.56.204 7001
1846:X 13 Nov 20:23:07.491 * +slave slave 192.168.56.206:7003 192.168.56.206 7003 @ stn-master 192.168.56.204 7001
1846:X 13 Nov 20:23:09.532 # +sdown slave 192.168.56.206:7003 192.168.56.206 7003 @ stn-master 192.168.56.204 7001


새롭게 선출된 마스터에 접속
ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.56.203,port=7000,state=online,offset=9045,lag=1
slave1:ip=192.168.56.205,port=7002,state=online,offset=9333,lag=0
master_repl_offset:9333
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:9332
192.168.56.204:7001>

죽었던 마스터 재시작
ubuntu@ubuntu:~/redis-3.2.5/src$  sudo ./redis-server /etc/redis/7003.conf
1846:X 13 Nov 20:26:17.859 # -sdown slave 192.168.56.206:7003 192.168.56.206 7003 @ stn-master 192.168.56.204 7001
1846:X 13 Nov 20:26:27.809 * +convert-to-slave slave 192.168.56.206:7003 192.168.56.206 7003 @ stn-master 192.168.56.204 7001

ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> info replication
# Replication
role:master
connected_slaves:3
slave0:ip=192.168.56.203,port=7000,state=online,offset=46026,lag=0
slave1:ip=192.168.56.205,port=7002,state=online,offset=46026,lag=0
slave2:ip=192.168.56.206,port=7003,state=online,offset=46026,lag=0
master_repl_offset:46026
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:46025

```

## 6.Pub/Sub
Pub/Sub 시스템에서는 채널에 구독 신청을 한 모든 subscriber에게 메시지를 전달한다. "던지는" 시스템이기 때문에 메시지를 보관하지 않는다.
```
ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> info replication
# Replication
role:master
connected_slaves:3
slave0:ip=192.168.56.206,port=7003,state=online,offset=155,lag=1
slave1:ip=192.168.56.203,port=7000,state=online,offset=155,lag=1
slave2:ip=192.168.56.205,port=7002,state=online,offset=155,lag=1
master_repl_offset:155
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:154

SUBSCRIBE
^Cubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.203 -p 7000 -a testserver
192.168.56.203:7000> SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1

ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1

^Cubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.205 -p 7002 -a testserver
192.168.56.205:7002> SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1

PUBLISH
^Cubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> PUBLISH testChannel "redis pub sub test"
(integer) 1


아래 와 같이 메시지를 받게 된다.
^Cubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.203 -p 7000 -a testserver
192.168.56.203:7000> SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1
1) "message"
2) "testChannel"
3) "redis pub sub test"

ubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.204 -p 7001 -a testserver
192.168.56.204:7001> SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1
1) "message"
2) "testChannel"
3) "redis pub sub test

^Cubuntu@ubuntu:~/redis-3.2.5/src$ sudo ./redis-cli -h 192.168.56.205 -p 7002 -a testserver
192.168.56.205:7002> SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1
1) "message"
2) "testChannel"
3) "redis pub sub test"
```

출처 :
http://redis.io
http://bryan.wiki/244
http://zuriyang.tistory.com/136
http://www.joinc.co.kr/w/man/12/REDIS/IntroDataType
