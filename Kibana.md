# Kibana - visualization





Kibana는 시각화 기술로서 Elasticsearch에 의해 색인된 데이터를 바탕으로 히스토그램, 막대그래프, 파이차트를 제작하고 위치데이터, 시계열 분석, 그래프관계 탐색 등을 지원합니다.

**Kibana를 통해 구상한 예시 Dashboard**를 살펴보겠습니다.



![kibana_dashboard_blur](C:\Users\hcwan\Desktop\secondary task\홈페이지 관리\access_log프로젝트\kibana_dashboard_blur.JPG)



필자가 사용한 kibana-oss-7.10.2 버전에서 기본으로 제공하는 시각화 양식은 위 9가지에 +a 가 있으며, 이에 대한 자세한 설명은 아래 [kibana 사용하기] 편에서 다루겠습니다.

그래프를 그리는 기본적인 방법(x축, y축 설정)과 유사하며 여러가지 옵션을 제공합니다. 시간, 위치, 횟수 등의 데이터를 조합해 다양한 시각화 자료를 만들고 조합할 수 있습니다. 단 이를 위해 색인된 데이터의 field 및 type, 구조 등을 손봐야하며 이는 Elasticsearch를 통해 가능하지만, 한번 색인된 데이터의 구조를 수정하는 것은 어렵기 때문에 Elasticsearch에서 설명한 index template을 활용하거나 Logstash의 filter plugin을 이용해 데이터를 정제하는 것을 추천합니다.

Kibana는 간단하게 설치 및 설정이 가능하며, 기본으로 제공되는 sample data를 활용해 kibana를 실제로 사용하면서 내용을 알아보겠습니다.



### 1. Kibana 설치하기 

```bash
# kibana 설치
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-oss-7.10.2-linux-x86_64.tar.gz
$ tar -zxvf kibana-oss-7.10.2-linux-x86_64.tar.gz
```



```bash
# 환경 설정 수정 (kibana 디렉토리에서)
$ vi config/kibana.yml
-----------------------
server.port: 5601
server.host: "[kibana실행 host의 ip주소]"
elasticsearch.hosts: ["elasticsearch실행 ip주소:9200"]
-----------------------
```



```bash
# kibana 실행
$ bin/kibana &		# & : 백그라운드 실행
```





