# Filebeat - Lightweight shipper for logs





Filebeat는 **경량화된 로그 수집기**로 보안 장치, 클라우드, 컨테이너, 호스트 또는 OT 등에서도 수집이 가능합니다. 컨테이너 및 클라우드 환경에서도 실행되기 때문에 Kubernetes, Docker, 클라우드 배포에서 Filebeat를 배포하고 사용할 수 있다. 로그를 `1 Line`씩 읽고 전달하며, 시스템이 중단되더라도 마지막 중단점을 기억해 재가동시 이어서 전달을 시작한다.



![filebeat architecture](https://www.elastic.co/guide/en/beats/filebeat/current/images/filebeat.png)



Filebeat는 **`Input`**, **`Harvester`**, **`Spooler`** 라는 3가지 주요 구성요소를 갖는다. 

`Input`은 데이터 소스를 찾고 harvester를 통제한다. 데이터 소스 경로에서 설정된 type 및 정보와 일치하는 파일을 찾고 그 파일을 읽기 위해 harvester를 실행한다. 

하나의 `Harvester`는 하나의 파일을 담당하고 데이터 파일을 열고 닫으며 로그 데이터를 `1 Line` 단위로 읽는다. 즉, 읽을 데이터 파일이 많다면 그에 따라 harvester의 개수도 많아진다. 

`spooler`는 harvestor가 읽은 로그 데이터를 집계하고 연결된 목적지로 전달한다.



Filebeat는 다수의 Input과 Harvester로 구성된다. Input 대상이 되는 파일을 어디까지 읽었는지, 어디까지 전송했는지에 대한 정보는 Harvester의 offset에 주기적으로 저장되며 이는 registry 파일에서 관리된다. elasticsearch, logstash, 카프카, 레디스 등 output 미들웨어 시스템에 문제가 발생하면, filebeat는 마지막으로 보낸 행을 기록하고, 문제가 해결될때까지 계속 데이터를 수집한다. 이러한 관리 덕에 filebeat를 정지하고 실행해도 데이터의 위치를 기억한 상태로 기동된다. Harvester는 데이터를 output에 연결된 목적지로 보낸 후에 데이터를 잘 받았다는 응답을 기다리는데, 해당 응답을 받지 못할 경우 다시 데이터를 보내게됨으로써 반드시 한번은 손실없이 데이터를 보낼 수 있다.





**특징**

* BackPressure-sensitive 프로토콜 : 연결된 Logstash 또는 Elasticsearch의 데이터 처리 속도에 맞춰 Filebeat의 읽기 속도를 조절함
* Autodiscover : 자동 검색을 통해 새로운 컨테이너를 감지하고 적합한 Filebeat 모듈을 통해 동적으로 컨테이너를 모니터링함
* 



1. filebeat는 inode를 저장해 파일을 식별한다. 그러므로 inode를 조작하면 파일이 이미 처리되었는지 여부를 알기 어렵다.

2. filebeat는 전송한 파일에 대해 내부적으로 offset 정보를 저장하기 때문에 중간에 crash(장애, 통신지연 등)가 발생해도 이어서 전송이 가능하다. (디버그 모드로 실행시 offset확인 가능 $ filebeat -c filebeat.yml -e -d "*")

3. 거의 실시간으로 새 로그 행을 읽을 수 있도록 파일 핸들러를 열어두기 때문에 많은 수의 파일을 수집하는 경우, 열려있는 파일 수가 문제가 될 수 있다. registry_file에 각 파일의 상태를 유지하고 파일 상태를 통해 filebeat가 다시 시작될때 이전 위치에서 파일 읽기를 계속할 수 있다. 만약 많은 수의 새 파일이 생성되면 registry_file의 크기가 너무 커질 수 있다. 크기를 줄이기 위해 clean_removed와 clean_inactive 구성 옵션을 사용할 수 있다.
4. filebeat는 개행 문자를 사용해 이벤트의 끝을 감지하기 때문에 수집 중인 파일의 끝에 개행이 없을 경우 마지막 행을 읽지 않으므로 주의한다.
5. filebeat의 cpu 점유율이 높다면 파일을 너무 자주 검색하도록 설정된 것은 아닌지 filebeat.yml파일의 scan_frequency값을 확인해 1 미만인 경우 조정한다.



Linux OS에서 Filebeat는 inode와 device값을 사용해 파일을 식별한다. 디스크에서 파일을 제거하면 파일 순환에 의해 제거된 inode 값과 동일한 값을 새로 생성된 파일이 할당받을 수 있다. 이 경우에 Filebeat는 registry에 기록된 정보에 의해 동일한 파일로 간주하고 이전 위치에서 읽으려고 시도하기 떄문에 문제가 될 소지가 있다. 기본적으로 registry에 기록된 상태 정보는 절대 제거되지 않기 떄문에 inode 재사용 문제를 해결하려면 `clean_*` 옵션, 특히 `clean_inactive`를 사용해 비활성 파일의 상태를 제거하는 것이 좋다. 디스크에서 제거된 파일에 대해서는 `clean_removed`를 사용할 수 있다. `clean_removed`는 스캔 중에 파일을 찾을 수 없을 때마다 registry에서 파일 상태를 정리하고, 나중에 동일한 파일이 생성되면 처음부터 다시 전송된다.





> Logstash의 output은 @metadata 필드를 자동으로 제거하기 때문에 필드를 보존하고자 할 경우, mutate filter를 사용해 필드의 이름을 변경해주어야 한다.





**< Beats 제품군 >**

* Filebeat	:	로그 파일
* Metricbeat	:	메트릭
* Packetbeat	:	네트워크 데이터
* Winlogbeat	:	Windows 이벤트 로그
* Auditbeat	:	감사 데이터
* Heartbeat	:	가동 시간 모니터링
* Functionbeat	:	서버를 사용하지 않는 수집기

이외에도 오픈 소스 Beats들이 존재하며 공통 라이브러리인 libbeat를 기반으로 제작되었습니다. [Beats 커뮤니티 목록](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html)에서 원하는 기능의 Beats를 확인할 수 있습니다.





ref) 

https://yongho1037.tistory.com/709

https://coding-start.tistory.com/187











Elasticsearch 사용을 위한 mechinsm 설명은 여기까지 하고, 이제 실제 사용을 위한 설치 작업으로 들어가보겠습니다.



### 1. Filebeat 설치하기

```bash
# elasticsearch 설치
$ wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-oss-7.10.2-linux-x86_64.tar.gz
$ tar -zxvf filebeat-oss-7.10.2-linux-x86_64.tar.gz
```



```bash
# 환경 설정 수정 (filebeat 디렉토리에서) - indent주의!!
$ mkdir config
$ vi config/temp.yml
-----------------------------
filebeat.inputs:		# 7버전 기점으로 filebeat.prospectors -> filebeat.inputs 변경 
- type: log				# 6버전 기점으로 input_type -> type 변경 (type에 맞는 확장자 사용)
  enabled: true
  paths:
    - /home/paulmauer/log*		# 수집할 log 파일의 경로
  tail_files: true				# filebeat 시작시점을 기준으로 파일 끝에서부터 로깅을 읽기 시작
  ignore_older: 1m				# filebeat 시작시점을 기준으로 설정한 시간 이전의 내용은 무시
  close_inactive: 2m
  clean_inactive: 15m
  
# filebeat logging 설정 
logging.level: info
logging.to_files: true
logging.files:
  path: /home/paulmauer/elastic/filebeat-7.10.2-linux-x86_64/logs/
  name: filebeat-log
  keepfiles: 7
  rotateeverybytes: 524288000
  
# 수집된 데이터를 logstash 전송
output.logstash:
  hosts: ["logstash실행서버IP:5044"]
-----------------------------
```



```bash
# filebeat 실행
$ ./filebeat -c config/access_log.yml -d publish &		# & : 백그라운드 실행
```



