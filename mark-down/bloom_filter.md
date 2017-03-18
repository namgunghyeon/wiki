# Bloom_Filter정리
## Bloom_Filter란
데이터가 있는지 없는지 확인할 수 있는 간단한 자료구조로 데이터가 없는데 있다고 할 수 는 있지만, 없는걸 있다고 할 수 는 없다.

데이터를 여러개의 Hash Function으로 생성된 값들을 인덱스로 하고 값은 1로 마킹한다.
Hash된 값들이 모두 1이여야 데이터가 있다고 판단하고, 하나라도 1일 아니면 없다고 판한다.

Hash Function은 충돌 가능성이 있어, 없는걸 있다고**(false positive)** 할 수 있다.

**False Positive**
문제가 되는 False Positive를 조절할 수 있는 방법으로는
해쉬 함수를 늘려 충돌을 회피한다.

![image](https://github.com/namgunghyeon/wiki/blob/master/images/bloom-filter/bloom-filter_1.png?raw=true)


**위 자료구조의 특성상 삭제는 할 수 없다**.

## 사용되는 곳
크롬 악성 URL체크
Hadoop
Cassandra
BigTable


**출처:
https://namu.wiki/w/%EB%B8%94%EB%A3%B8%20%ED%95%84%ED%84%B0
http://itbrain.tistory.com/entry/Bloom-filter%EB%B8%94%EB%A3%B8-%ED%95%84%ED%84%B0
http://d2.naver.com/helloworld/749531
