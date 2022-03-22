# Elasticsearch - search engine





Elasticsearch는 분산형 RESTful 검색 및 분석 엔진입니다. 정형, 비정형, 위치정보, 메트릭 등 다양한 유형의 검색을 수행하고 결합할 수 있습니다. 크게 **indexing**(색인), **analyzing**(분석), **searching**(검색) 등의 기능이 존재하며 색인된 데이터를 source로 사용하여 분석과 검색을 실시합니다. 

**Elasticsearch의 작동 Mechanism**에 대해 살펴보겠습니다. (cluster, node 등에 관한 시스템 구조는 향후에 서술)



![Elasticsearch Mechanism](C:\Users\hcwan\AppData\Roaming\Typora\typora-user-images\image-20220127154025683.png)



수집한 데이터를 Elasticsearch를 통해 **indexing**해줍니다. 이 때 **Index Templates**을 미리 작성해두면 index의 field를 설정하거나 data 속성을 정해줄 수 있습니다. 한번 색인되어 만들어진 index data는 수정이 어렵기 때문에 원하는 목적을 고려해 미리 Index Templates을 작성해 이용하는 것이 좋습니다. 그 결과 만들어진 index data를 resourece로 사용하여 **analyzing**, **searching** 작업을 수행합니다. (색인된 데이터는 향후 Kibana를 통해 시각화 가능)

> Index Templates 참고) https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html

Elasticsearch 사용을 위한 mechinsm 설명은 여기까지 하고, 이제 실제 사용을 위한 설치 작업으로 들어가보겠습니다.



### 1. Elasticsearch 설치하기

root 권한으로 실행시 오류가 발생할 수 있으므로, 그때는 다른 계정을 활용해 elasticsearch의 소유권을 변경해주시고 구동하시기 바랍니다.

```bash
# elasticsearch 설치
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-7.10.2-linux-x86_64.tar.gz
$ tar -zxvf elasticsearch-oss-7.10.2-linux-x86_64.tar.gz
```



```bash
# 환경 설정 수정 (elasticsearch 디렉토리에서)
$ vi config/elasticsearch.yml
-----------------------------
network.host: 0.0.0.0			# 모든 ip 접근 가능
discovery.type: single-node		# 7버전부터 single-node를 생략하면 bootstrap fail
discovery.seed_hosts:
    - 192.168.1.10:9300
    - 192.168.1.11
```

seed_hosts 참고) https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-hosts-providers.html



```bash
# elasticsearch 실행
$ bin/elasticsearch -d		# -d : 백그라운드 실행

# 실행확인
$ curl -i http://localhost:9200
```



---



##### [ Troubleshooting ]

**ERROR: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]**

```bash
# vm.max_map_count 설정(지속)
$ sudo vi /etc/sysctl.conf
--------------------------
vm.max_map_count=262144
--------------------------

$ sudo sysctl -p
----------------
vm.max_map_count = 262144

$ cat /proc/sys/vm/max_map_count
--------------------------------
262144

# 시스템에 바로 설정도 가능(단, 재부팅시 초기화됨)
$ sudo sysctl -w vm.max_map_count=262144
```



