# InfluxDB정리
## 1.InfluxDB란
Time-series DB는 데이터베이스 중에서도 시계열 데이터를 저장하고 활용하는데 특화된 형태의 데이터베이스이다.
- InfluxDB는 과거에 data store를 위해 구글이 만든 key/value database libary를 사용했었고 현재는 자체적으로 만든 TSM Engine으로 변경했다.
  - leveldb - a fast and lightweight key/value DB libary
  - TSM(Time Structured Merge) Engine https://www.influxdata.com/new-storage-engine-time-structured-merge-tree
- SQL-like Query Language를 지원한다. group by, join 복수개의 time series를 Merge도 가능한다.
- Cluster에 새로운 Node만 추갛면 쉽게 scale-out할 수 있다.
- 지정된 시간마다 Query를 실행하고 실행된 결과를 저장할 수 있다.(crontab 느낌)
- HA는 relay https://docs.influxdata.com/influxdb/v1.1/high_availability/relay/ 를 통해 가능하다.
- Go언어로 만들어져 있다.

## 2. 설치
https://docs.influxdata.com/influxdb/v1.1/introduction/installation/


## 3. Realy

https://github.com/influxdata/influxdb-relay/blob/master/README.md


작업 중
