# Elasticsearch의 세부 개념



먼저 Elasticsearch에서 색인된 데이터의 구조와 RDBMS(관계형 DB)의 데이터 구조를 비교해보겠습니다.



![db_elasticsearch_1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile22.uf.tistory.com%2Fimage%2F998444375C98CC021F2221)



|          관계형 DB | Elasticsearch        |
| -----------------: | :------------------- |
|           Database | Index                |
|              Table | Type                 |
|                Row | Document             |
|             Column | Field                |
|              Index | Analyze              |
|        Primary key | _id                  |
|             Schema | Mapping              |
| Physical partition | Shard                |
|  Logical partition | Route                |
|         Relational | Parent/Child, Nested |
|                SQL | Query DSL            |





Elasticsearch Architecture

![elasticsearch_cluster](https://t1.daumcdn.net/cfile/tistory/99A97A355C98D42D2E)



* 클러스터(cluster)란? 

클러스터는 Elasticsearch의 가장 큰 시스템 단위이며 하나 이상의 노드로 이루어진 노드들의 집합이다. 클러스터는 데이터 접근 및 교환에 있어서 각각의 독립적인 시스템이며, 여러 대의 서버가 하나의 클러스터를 구성할 수도 있다. 물론 하나의 서버에 여러 개의 클러스터가 존재할 수도 있다.



* 노드(node)

Elasticsearch를 구성하는 하나의 단위 프로세스를 의미한다. 역할에 따라 Master-eligible, Data, Ingest, Tribe 노드로 구분할 수 있다. 

 		** master-eligible node

​		 ** Data node

​		 ** Ingest node

​		 ** Coordination only node



* 인덱스(Index)

인덱스는 관계형 DB의 database와 대응되는 개념이다. 



* 샤드(Shard)

샤딩(Sharding)은 데이터를 분산해서 저장하는 방법이다. elasticsearch에서 스케일 아웃을 위해 index를 여러 개의 shard로 쪼개어 보관할 수 있습니다. 기본적으로 1개가 존재하며, 검색 성능 향상을 위해 클러스터의 샤드 갯수를 조절하는 튜닝을 하기도 한다.



* 복제(Replica)

shard와 유사한 형태로, 노드를 잃었을 경우 데이터의 안정성을 위해 shard를 복사해둔 것이다. 그러므로 replica는 서로 다른 노드에 존재하는 것을 권장하며 아래 그림에서 서로 다른 노드에 replica가 존재하는 것을 알 수 있다.



![node](https://t1.daumcdn.net/cfile/tistory/991563425C98CB341A)





Elasticsearch의 특징

1. scale out - shard를 사용해 수평적 규모를 확장할 수 있다
2. 데이터 안정성 - replica를 통해
3. Schema free - json으로 데이터 검색을 수행하기 때문에 스키마 개념이 존재하지 않음

4. HTTP Restful API 활용

| DB CRUD | Elasticsearch Restful API |
| ------: | ------------------------- |
|  SELECT | GET                       |
|  INSERT | PUT                       |
|  UPDATE | POST                      |
|  DELETE | DELETE                    |



 

**사용**

document 생성

```bash
$ curl -XPOST 'localhost:9200/indexName/blog/1?pretty' -d '{"postName" : "elasticsearch", "catecory" : "IT"}' -H 'Content-Type: application/json'

# indexName : 인덱스명
# blog : 타입
# id : 1
# -d 옵션 : 데이터를 json포맷으로 전달		-H 옵션 : 헤더		?pretty : 결과를 예쁘게 보여주기

---------------------------------------------------------------
{
  "_index" : "indexName",
  "_type" : "blog",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
---------------------------------------------------------------
```



document 조회

```bash
$ curl -XGET 'localhost:9200/indexName/blog/1?pretty'
-------------------------------------------------------
{
  "_index" : "indexName",
  "_type" : "blog",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "postName" : "elasticsearch",
    "category" : "IT"
  }
}
```





Elasticsearch를 능숙하게 사용하기 위해서 Restful API(GET, PUT, POST, DELETE) 사용에 익숙해야 한다. queryDSL을 통해 쿼리를 작성하고 사용자가 원하는 방식으로 사용할 수 있다.











ref) https://victorydntmd.tistory.com/308























