# Cassandra정리
## 1.Cassandra란
Cassandra는 Scalability와 High Avaliablility에 최적화된 대표적인 분산형 Data Storeage이다. Consistent hashing을 이용한 Ring 구조와 Gossip Protocol을 구현했으며, 각 노드 장비들의 추가, 제거 등이 자유롭고, 데이터 센터까지 고려 할 수 있는 데이터 복제 정책을 사용하여 안정성 축면에서 많은 장점이 있다.
하지만 Join이나 Transaction을 지원하지 않고, Index등의 검색을 위한 기능도 매우 단출한다. Keyspace나 Table등을 과도하게 생성할 경우 Memory Overflow가 발생할 수 있음을 고려해야한다.


## 2.구조

Partition Key데이터의 hash값을 기준으로 Data를 분산한다. 처음 각 노드가 Ring에 참여하면 Cassandra의 conf/cassandra.yaml에 정의된 각 설정을 통해 각 노드마다 고유의 Hash 범위를 부여 받는다. Read/Write요청이 오면 Partition Key를 통해서 Hash값을 계산해 데이터가 어느 노드에 있는지 알고 접근할 수 있다. 계산된 Hash값을 Token이라고 부른다.

초창기의 Cassandra Data Structure는 **Keyspace > Column Family > Row > Column(Column Name + Column Value)** 형태로 구성되어 있었다.
Keyspace와 Column Family에 대한 정보는 모든 Cassandra Node의 메모리에 저장된다

Cassandra 1.2에서는  **Keyspace > Table > Row > Column(Column Name + Column Value)** 로 변경이 되었다.



여기에서의 Row와 Column은 Cassandra Data Layer의 Row와 Column을 의미하지 않는다.

**Keyspace**

**Table**

**Row**

**Column**

**Virtual Node**
Cassandra에 Data를 CURD를 진행하고. 해당 데이터는 Partition Key로 지정된 Column의 Value가 Row Key가 되고 이 Row Key를 Hashing하여 Token을 계산한 뒤, 해당 Token의 범위에 속한 Node를 찾아 CURD를 진행한다. 각 노드별 Token의 범위가 할당되어 있어야한다.

하나의 노드에서 연속한 3개의 Toekn범위에 대한 저장 의무를 가진다.

Cassandra 1.2부터 Virtual Node라는 기능이 추가되었다.


하나의 노드의 옵션에서 설정한 만큼 다수개의 가상노드와 가상노드 각각이 정해진 Token범위에 대한 저장 의무를 가진다.
Cassandra 노드안에 가상 노드를 여러개 두고, 아주 잘게 나누어진 token 범위를 가상 노드들에게 할당하여 데이터를 분산하는 개념 Virtual Node을 두어서 데이터를 좀 더 균일하게 분산하고 여러곳에 분산되어 있어 노드의 추가, 제거시 데이터의 이동, 복제, 리밸런싱에 높은 성능향상을 가져다 준다.
conf/cassandra.yaml의 num_tokens의 항목으로 조정할 수 있다.


## 3.용어
##### Partition key
Cassandra에서 Data를 분산 저장하기 위한 Unique한 Key
Partition Key는 Table를 구성할 때 반드시 1개 이상이 지정되어야 하며, 여러 개 지정될 수 있다.
Partition Key가 하나일 경우 해당 Partition Key로 지정된 CQL Column의 Value가 실제 Cassandra Data Layer의 Row Key가 된다.
Partition Key가 여러 개일 경우 :문자와 함께 조합한 값들이 실제 Cassandra Data Layer의 Row Key가 된다.

##### Cluster Key
Cluster Key는 정렬에 대한 기준 역활을 담당한다.

##### Primary Key
Primary Key는 CQL table에서의 각 Row를 각자 Unique 하게 결졍해누즞 기준 역할을 담당한다.
Primary Key는 최소 1개 이상이 Partition Key와 0개 이상의 Cluster Key로 구성된다.

##### Composite Key(=compound key)
1개 이상의 CQL Column들로 이루어진 Primary Key를 Composite Key혹은 Composite key라고 부른다.

##### Composite Partition Key
Composite partition Key는 2개 이상의 다수의 CQL Column으로 이루어진 Partition Key를 의미한다.


**출처:
http://meetup.toast.com/posts/58**
