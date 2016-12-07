# kubernetes
## 1.kubernetes란(그리스어 배의 조타수)
Container Orchestration툴
구글에서 만든 Doker Orchestrator 구글 사내에서 비공개로 쓰던 Container기술(Go언어로 개발) [borg](http://research.google.com/pubs/pub43438.html)
현재 Hot한 오픈소스

 - 여러 Host를 묶어 클러스터를 구성
 - Container를 적절한 위치에 배포
 - Container가 죽으면 자동으로 복구
 - Scaling, Replication, Rolling update, Rollback

![kubernetes](https://raw.githubusercontent.com/namgunghyeon/wiki/93128abd9e1b97a9202ff7f8f231d0cdaf2bfd77/images/kubernetes/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-12-08%20%EC%98%A4%EC%A0%84%2012.13.38.png)

![kubernetes](https://raw.githubusercontent.com/namgunghyeon/wiki/93128abd9e1b97a9202ff7f8f231d0cdaf2bfd77/images/kubernetes/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-12-08%20%EC%98%A4%EC%A0%84%2012.13.26.png)


##### Docker+Kubernetes를 이용한 빌드 서버 가상화 사례 SK플래닛@tech세미나
https://www.youtube.com/watch?v=EZE-aw2ndjg

### Cluster
```
어플리케이션 컨테이너들이 배치되고 실행되는 컴퓨팅 자원
```
### Pod
```
어플리케이션 실행에 필요한 도커 컨테이너들의 집합
```
### Minion
```
관리를 받는 노드. 도커가 설치되어 있으며 설제로 컨테이너들이 생성되는 서버
```
### Replication Controller
```
pod들의 라이플사이클 관리자
pod들의 Health Status를 체크
```
### Service
```
pod들을 하나의 서비스로 외부에 접근할 수 있도록 추상화한다.
pod들을 하나의 서비스로 묶어서 외부에 제공한다.
```
### Label
```
pod들의 그룹핑/조직화를 위해 관리하는 key-value pair/메타데이터
```

작성중

출처:
http://www.slideshare.net/tommylee98229/2-kubernetes
http://www.popit.kr/kubernetes-introduction/
http://ingeec.tistory.com/74
http://brownbears.tistory.com/1
http://ggoals.tistory.com/36
https://giljae.com/kubernetes%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-c836675c5c7#.3ait93bx1
