# Elastic

source: `{{ page.path }}`



데이터과학이 발전하면서 데이터 기술 역시 함께 성장하고 있다. 데이터 기술의 영역에는 **저장, 정제, 색인, 분석, 시각화** 등 다양한 종류가 있으며 최근 **Elastic**사가 주목받고 있다. Elastic의 솔루션 중 분산 검색엔진으로 주목받고 있는 **Elasticsearch**는 자바의 검색 기술 라이브러리인 아파치 루신(Apache Lucene)을 기반으로 탄생했다. 이외에도 데이터 처리 파이프라인인 **Logstash**, 시각화 기술의 **Kibana** 등이 존재하며, 가장 인기 있는 3가지 제품을 흔히 'ELK 스택'이라고 부른다.

> ELK 스택 : Elasticsearch(분석 및 색인), Logstash(수집 및 정제), Kibana(시각화)



Elastic의 제품은 확장성이 뛰어나고 쉽게 설치가 가능하다. 처음에 작은 규모로 Elastic solution을 적용해도 점차 확장할 수 있으며 API 등을 통해 구조를 단순화할 수 있다. Elastic은 이상 데이터 및 외부 공격을 감지하는 기능인 프리러트(Prelert)를 강조하고 있으며 머신러닝과 결합한 것이 특징이다.



---



#### 내 사용자 개발 환경

```bash
OS: ubuntu 20.04 LTS
JDK: java-11-openjdk
Elstic: OSS-7.10.2
apt repository 최신화
```

