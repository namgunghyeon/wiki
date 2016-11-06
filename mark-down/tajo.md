# Tajo정리
## 1.Tajo란
하둡 기반의 대용량 데이터 웨어 하우스 시스템 SQL 표준, 전체 질의 분산 처리
TajoMaster와 TajoWorker로 구성되며, 마스터에 문제가 생길 시 정상적으로 사용할 수 있도록 슬레이브 서버에 QueryMaster를 두어 정상적으로 처리가
가능하도록 구성되어 있다.
![tajo](https://github.com/namgunghyeon/wiki/blob/master/images/tajo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-14%20%EC%98%A4%EC%A0%84%2012.33.44.png?raw=true)

**클라이언트**
- tsql
- JDBC드라이버

**타조 마스터**
- 타조 클러스터의 마스터 역활
- 클라이언트 API를 RPC로 제공
- 질의를 파싱해서 논리적인 질의 실행 계획을 수집하는 플래너를 제공
- 수립된 질의 최적화
- 쿼리 마스터를 관리. 질의 실행 요청이 들어올 경우, 유효한 타조워커 중에 하나를 쿼리 마스터로 선정
- 전체 클러스터의 자원을 관리


**쿼리 마스터**
- 타조 마스터 생성한 논리 실행 계획을 분산 실행 계획으로 변환, 변환된 분산 실행 계획을 제어하고, 태스크를 스케쥴링한다.
- 질의 하나당 쿼리 마스터 하나가 생성된다.

**타조 워커**
- 타조워커는 타조워커와 쿼리마스터 두 가지 역활을 수행한다.


## 2.질의 실행 계획 과정
사용자가 작성한 질의 대한 구분 분석을 요청, 질의 구문 분석기는 클라이언트가 전달한 질의 대수학 표현식으로 생성, JSON형태
![tajo](https://github.com/namgunghyeon/wiki/blob/master/images/tajo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-14%20%EC%98%A4%EC%A0%84%201.15.42.png?raw=true)


## 3.설치
http://blrunner.com/101 참고해서 설치할 예정
![tajo](https://github.com/namgunghyeon/wiki/blob/master/images/tajo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-10-25%20%EC%98%A4%EC%A0%84%2012.22.52.png?raw=true)


계정 생성
```
하둡이 설치된 서버에 타조 계정 생성
192.168.56.107 hadoop-master-1
192.168.56.108 hadoop-master-2
192.168.56.109 hadoop-slave-01
192.168.56.110 hadoop-slave-02
192.168.56.111 hadoop-slave-03

adduser tajo
su - tajo
```

타조 다운
```
http://apache.tt.co.kr/tajo/tajo-0.11.3/tajo-0.11.3.tar.gz
```

conf/tajo-env.sh 설정
```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export HADOOP_HOME=/home/hadoop/hadoop-2.7.3
export TAJO_MASTER_HEAPSIZE=1000
export TAJO_WORKER_HEAPSIZE=5000
export TAJO_PID_DIR=/home/tajo/pids
export TAJO_LOG_DIR=/home/tajo/logs
```

conf/tajo-site.xml 설정
```

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

<property>
  <name>tajo.rootdir</name>
  <value>hdfs://hadoop-master-1:9000/tajo</value>
</property>

<property>
  <name>tajo.master.umbilical-rpc.address</name>
  <value>hadoop-master-1:26001</value>
</property>


<property>
  <name>tajo.master.client-rpc.address</name>
  <value>hadoop-master-1:26002</value>
</property>

<property>
  <name>tajo.resource-tracker.rpc.address</name>
  <value>hadoop-master-1:26003</value>
</property>

<property>
  <name>tajo.catalog.client-rpc.address</name>
  <value>hadoop-master-1:26005</value>
</property>

<property>
  <name>tajo.worker.resource.dfs-dir-aware</name>
  <value>true</value>
</property>

</configuration>
```

conf/workers
```
hadoop-slave-01
hadoop-slave-02
hadoop-slave-03
```

설정된 정보를 압축
```
tar cvfz tajo.tar.gz tajo-0.11.3

복사된 타조 압축 파일을 slave에게 복사
scp tajo.tar.gz tajo@hadoop-master-2:/home/tajo
scp tajo.tar.gz tajo@hadoop-slave-01:/home/tajo
scp tajo.tar.gz tajo@hadoop-slave-02:/home/tajo
scp tajo.tar.gz tajo@hadoop-slave-03:/home/tajo
```

실행
```
타조 마스터에서 /home/tajo/tajo-0.11.3
./bin/start-tajo.sh
```

Tajo master web UI: http://192.168.56.107:26080
Tajo Client Service: 192.168.56.107:26002


tsql 실행
```
/home/tajo/tajo-0.11.3/bin
bash tsql
```

샘플데이터 다운
```
/home/tajo/data

mkdir data
cd data
wget http://files.grouplens.org/datasets/movielens/ml-1m.zip
unzip ml-1m.zip
```

tsql로 샘플데이터 업로드
```
tajo@hadoop-master-1:~/tajo-0.11.3/bin$ bash tsql

default> \dfs -mkdir -p /movielens/movies
default> \dfs -mkdir -p /movielens/ratings

default> \dfs -put /home/tajo/data/ml-1m/movies.dat /movielens/movies
default> \dfs -put /home/tajo/data/ml-1m/ratings.dat /movielens/ratings

 \dfs -ls /movielens
```

데이터 베이스 및 테이블 생성
```
default> create database movie_lens;
OK
default>  \c movie_lens;
You are now connected to database "movie_lens" as user "tajo".

movie_lens> CREATE EXTERNAL TABLE ratings (
 user_id INT ,
 movie_id INT ,
 rating INT ,
 rated_at INT
) USING TEXT WITH ('text.delimiter'='::') LOCATION 'hdfs://hadoop-master-1:9000/movielens/ratings/';

movie_lens> CREATE EXTERNAL TABLE movies (
  movie_id INT,
  title TEXT,
  genres TEXT
) USING TEXT WITH ('text.delimiter'='::') LOCATION 'hdfs://hadoop-master-1:9000/movielens/movies/';

```

테스트 쿼리
```
movie_lens> select * from movies limit 5;
movie_id,  title,  genres
-------------------------------
1,  Toy Story (1995),  Animation|Children's|Comedy
2,  Jumanji (1995),  Adventure|Children's|Fantasy
3,  Grumpier Old Men (1995),  Comedy|Romance
4,  Waiting to Exhale (1995),  Comedy|Drama
5,  Father of the Bride Part II (1995),  Comedy
(5 rows, 0.639 sec, 0 B selected)

movie_lens> select count(*) from ratings;
Progress: 0%, response time: 3.276 sec
Progress: 0%, response time: 3.278 sec
Progress: 0%, response time: 3.679 sec
Progress: 100%, response time: 3.749 sec
?count
-------------------------------
1000209
(1 rows, 3.749 sec, 16 B selected)

movie_lens>  SELECT a.user_id, a.movie_id, b.title, a.rating, to_char(to_timestamp(a.rated_at), 'YYYY-MM-DD HH24:MI:SS') as rated_at
> from ratings a, movies b
> where a.movie_id = b.movie_id
> ;
Progress: 0%, response time: 0.948 sec
Progress: 0%, response time: 0.95 sec
Progress: 0%, response time: 1.351 sec
Progress: 0%, response time: 2.156 sec
Progress: 50%, response time: 3.158 sec
Progress: 50%, response time: 4.162 sec
Progress: 54%, response time: 5.164 sec
Progress: 54%, response time: 6.166 sec
Progress: 100%, response time: 7.122 sec
user_id,  movie_id,  title,  rating,  rated_at
-------------------------------
1,  1193,  One Flew Over the Cuckoo's Nest (1975),  5,  2001-01-01 07:12:40
1,  661,  James and the Giant Peach (1996),  3,  2001-01-01 07:35:09
1,  914,  My Fair Lady (1964),  3,  2001-01-01 07:32:48
1,  3408,  Erin Brockovich (2000),  4,  2001-01-01 07:04:35
1,  2355,  Bug's Life, A (1998),  5,  2001-01-07 08:38:11
1,  1197,  Princess Bride, The (1987),  3,  2001-01-01 07:37:48
1,  1287,  Ben-Hur (1959),  5,  2001-01-01 07:33:59
1,  2804,  Christmas Story, A (1983),  5,  2001-01-01 07:11:59
1,  594,  Snow White and the Seven Dwarfs (1937),  4,  2001-01-01 07:37:48
1,  919,  Wizard of Oz, The (1939),  4,  2001-01-01 07:22:48
1,  595,  Beauty and the Beast (1991),  5,  2001-01-07 08:37:48
1,  938,  Gigi (1958),  4,  2001-01-01 07:29:12
1,  2398,  Miracle on 34th Street (1947),  4,  2001-01-01 07:38:01
1,  2918,  Ferris Bueller's Day Off (1986),  4,  2001-01-01 07:35:24
1,  1035,  Sound of Music, The (1965),  5,  2001-01-01 07:29:13
1,  2791,  Airplane! (1980),  4,  2001-01-01 07:36:28
1,  2687,  Tarzan (1999),  3,  2001-01-07 08:37:48
1,  2018,  Bambi (1942),  4,  2001-01-01 07:29:37
1,  3105,  Awakenings (1990),  5,  2001-01-01 07:28:33
1,  2797,  Big (1988),  4,  2001-01-01 07:33:59
1,  2321,  Pleasantville (1998),  3,  2001-01-01 07:36:45
1,  720,  Wallace & Gromit: The Best of Aardman Animation (1996),  3,  2001-01-01 07:12:40
1,  1270,  Back to the Future (1985),  5,  2001-01-01 07:00:55
1,  527,  Schindler's List (1993),  5,  2001-01-07 08:36:35
1,  2340,  Meet Joe Black (1998),  3,  2001-01-01 07:01:43
1,  48,  Pocahontas (1995),  5,  2001-01-07 08:39:11
1,  1097,  E.T. the Extra-Terrestrial (1982),  4,  2001-01-01 07:32:33
1,  1721,  Titanic (1997),  4,  2001-01-01 07:00:55
1,  1545,  Ponette (1996),  4,  2001-01-07 08:35:39
1,  745,  Close Shave, A (1995),  3,  2001-01-07 08:37:48
1,  2294,  Antz (1998),  4,  2001-01-07 08:38:11
1,  3186,  Girl, Interrupted (1999),  4,  2001-01-01 07:00:19
1,  1566,  Hercules (1997),  4,  2001-01-07 08:38:50
1,  588,  Aladdin (1992),  4,  2001-01-07 08:37:48
1,  1907,  Mulan (1998),  4,  2001-01-07 08:38:50
1,  783,  Hunchback of Notre Dame, The (1996),  4,  2001-01-07 08:38:11
1,  1836,  Last Days of Disco, The (1998),  5,  2001-01-01 07:02:52
1,  1022,  Cinderella (1950),  5,  2001-01-01 07:00:55
1,  2762,  Sixth Sense, The (1999),  4,  2001-01-01 07:34:51
1,  150,  Apollo 13 (1995),  5,  2001-01-01 07:29:37
1,  1,  Toy Story (1995),  5,  2001-01-07 08:37:48
1,  1961,  Rain Man (1988),  5,  2001-01-01 07:26:30
1,  1962,  Driving Miss Daisy (1989),  4,  2001-01-01 07:29:13
1,  2692,  Run Lola Run (Lola rennt) (1998),  4,  2001-01-01 07:26:10
1,  260,  Star Wars: Episode IV - A New Hope (1977),  4,  2001-01-01 07:12:40
1,  1028,  Mary Poppins (1964),  5,  2001-01-01 07:29:37
1,  1029,  Dumbo (1941),  5,  2001-01-01 07:36:45
1,  1207,  To Kill a Mockingbird (1962),  4,  2001-01-01 07:11:59
1,  2028,  Saving Private Ryan (1998),  5,  2001-01-01 07:26:59
1,  531,  Secret Garden, The (1993),  4,  2001-01-01 07:35:49
1,  3114,  Toy Story 2 (1999),  4,  2001-01-01 07:36:14
1,  608,  Fargo (1996),  4,  2001-01-01 07:23:18
1,  1246,  Dead Poets Society (1989),  4,  2001-01-01 07:34:51
2,  1357,  Shine (1996),  5,  2001-01-01 06:38:29
2,  3068,  Verdict, The (1982),  4,  2001-01-01 06:43:20
2,  1537,  Shall We Dance? (Shall We Dansu?) (1996),  4,  2001-01-01 06:53:40
2,  647,  Courage Under Fire (1996),  3,  2001-01-01 06:49:11
2,  2194,  Untouchables, The (1987),  4,  2001-01-01 06:48:17
2,  648,  Mission: Impossible (1996),  4,  2001-01-01 06:58:33
2,  2268,  Few Good Men, A (1992),  5,  2001-01-01 06:48:17
2,  2628,  Star Wars: Episode I - The Phantom Menace (1999),  3,  2001-01-01 07:00:51
2,  1103,  Rebel Without a Cause (1955),  3,  2001-01-01 06:41:45
2,  2916,  Total Recall (1990),  3,  2001-01-01 06:56:49
2,  3468,  Hustler, The (1961),  5,  2001-01-01 06:35:42
2,  1210,  Star Wars: Episode VI - Return of the Jedi (1983),  4,  2001-01-01 06:29:11
2,  1792,  U.S. Marshalls (1998),  3,  2001-01-01 06:59:01
2,  1687,  Jackal, The (1997),  3,  2001-01-01 07:02:54
2,  1213,  GoodFellas (1990),  2,  2001-01-01 06:34:18
2,  3578,  Gladiator (2000),  5,  2001-01-01 06:42:38
2,  2881,  Double Jeopardy (1999),  3,  2001-01-01 07:00:02
2,  3030,  Yojimbo (1961),  4,  2001-01-01 06:33:54
2,  1217,  Ran (1985),  3,  2001-01-01 06:29:11
2,  3105,  Awakenings (1990),  4,  2001-01-01 06:37:53
2,  434,  Cliffhanger (1993),  2,  2001-01-01 07:02:54
2,  2126,  Snake Eyes (1998),  3,  2001-01-01 07:02:03
2,  3107,  Backdraft (1991),  2,  2001-01-01 07:00:02
2,  3108,  Fisher King, The (1991),  3,  2001-01-01 06:55:12
2,  3035,  Mister Roberts (1955),  4,  2001-01-01 06:37:05
2,  1253,  Day the Earth Stood Still, The (1951),  3,  2001-01-01 06:45:20
2,  1610,  Hunt for Red October, The (1990),  5,  2001-01-01 06:56:49
2,  292,  Outbreak (1995),  3,  2001-01-01 07:02:03
2,  2236,  Simon Birch (1998),  5,  2001-01-01 06:47:00
2,  3071,  Stand and Deliver (1987),  4,  2001-01-01 06:45:20
2,  902,  Breakfast at Tiffany's (1961),  2,  2001-01-01 06:41:45
2,  368,  Maverick (1994),  4,  2001-01-01 07:00:02
2,  1259,  Stand by Me (1986),  5,  2001-01-01 06:40:41
2,  3147,  Green Mile, The (1999),  5,  2001-01-01 06:37:32
2,  1544,  Lost World: Jurassic Park, The (1997),  4,  2001-01-01 07:02:54
2,  1293,  Gandhi (1982),  5,  2001-01-01 06:31:01
2,  1188,  Strictly Ballroom (1992),  4,  2001-01-01 06:53:40
2,  3255,  League of Their Own, A (1992),  4,  2001-01-01 06:48:41
2,  3256,  Patriot Games (1992),  2,  2001-01-01 06:57:19
2,  3257,  Bodyguard, The (1992),  3,  2001-01-01 07:01:13
2,  110,  Braveheart (1995),  5,  2001-01-01 06:37:05
2,  2278,  Ronin (1998),  3,  2001-01-01 06:58:09
2,  2490,  Payback (1999),  3,  2001-01-01 06:59:26
2,  1834,  Spanish Prisoner, The (1997),  4,  2001-01-01 06:40:13
2,  3471,  Close Encounters of the Third Kind (1977),  5,  2001-01-01 06:40:14
2,  589,  Terminator 2: Judgment Day (1991),  4,  2001-01-01 06:56:13
2,  1690,  Alien: Resurrection (1997),  3,  2001-01-01 07:00:51
```
