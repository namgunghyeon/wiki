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

## 4.HA

## 5.Cluster

## 6.Pub/Sub

출처 :
http://redis.io
http://bryan.wiki/244
http://zuriyang.tistory.com/136
http://www.joinc.co.kr/w/man/12/REDIS/IntroDataType
