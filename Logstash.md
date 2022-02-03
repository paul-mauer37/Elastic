# Logstash - collecting & preprocessing





Logstash는 수집 도구로 잘 알려진 데이터 처리 파이프라인(pipeline)입니다. 다양한 데이터 소스에서 데이터를 수집해 변환한 뒤 특정한 곳으로 데이터를 보내는 역할을 합니다. 파이프라인 내에서 데이터의 구문 분석 및 변환이 가능해 분석을 쉽고 빠르게 할 수 있도록 도와줍니다. 200개 이상의 플러그인을 지원해 다양한 기술과 조합해 사용할 수 있습니다.

**Logstash의 Architecture**에 대해 살펴보겠습니다.



![Logstash Architecture](https://cloudaffaire.com/wp-content/uploads/2020/02/word-image-29.png)



데이터를 수집해서 전달하는 하나의 파이프라인(pipeline) 형태이며 파이프라인은 **input - filter - output** 구조입니다. 다양한 데이터 소스로부터 `logstash input`을 통해 데이터가 수집되어 들어오며 해당 파이프라인의 `filter plugin`을 통과하면서 사용자의 의도에 알맞게 데이터가 정제됩니다. 정제된 데이터는 `logstash output`에 연결된 특정한 목적지(Destination)로 전달되며 Logstash의 수집 및 정제 절차가 완료됩니다. 

Logstash는 다중 파이프라인 구조를 갖기 때문에 동시에 여러 데이터 소스에서 데이터를 수집해 정제한 뒤 다양한 목적지로 보낼 수 있습니다.

이제 Logstash를 설치하고 사용자의 목적에 알맞게 파이프라인을 설계한 뒤, 실제로 수집 및 정제가 제대로 이루어지는지 확인해보겠습니다.



### 1. Logstash 설치하기 

```bash
# logstash 설치
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-oss-7.10.2-linux-x86_64.tar.gz
$ tar -zxvf logstash-oss-7.10.2-linux-x86_64.tar.gz
```



기본적인 logstash pipeline 사용을 위해 `stdin`, `stdout`으로 `input`, `output`을 설정한다. logstash를 실행한 터미널에서 `stdin`을 직접 입력하고 결과값으로 `stdout`을 받는다.

```bash
# 환경 설정 수정 (logstash 디렉토리에서)
$ vi config/temp.conf		# 신규 conf 파일 생성(pipeline 설계)
---------------------------
input { 
    stdin { } 
} 

filter {
#########내용########
}

output { 
    stdout { } 
}
--------------------------
```



```bash
# logstash 실행
$ bin/logstash -f config/temp.conf &		# -f : conf파일 지정   /   & : 백그라운드 실행

# output을 stdout으로 확인을 원할 경우
$ bin/logstash -f config/temp.conf
```





[ Troubleshooting ]   ~   test pipeline

##### ERROR: Failed to execute action {:action=>LogStash::PipelineAction::Create/pipeline_id:main, :exception=>"Java::JavaLang::IllegalStateException", :message=>"Unable to configure plugins:

* pipelind_id에 해당하는 파이프라인에서 오류 발생(pipeline_id는 main이 기본)

* 파이프라인에 사용한 플러그인 혹은 포트 문제일 수 있으므로 테스트를 하며 오류 수정

```bash
# logstash 실행 중에 config 파일 변경시 자동 reload(모든 설정에 대해 가능한 것은 아님)
$ vi config/logstash.yml
------------------------
config.reload.automatic: true
------------------------
```



```bash
# 테스트용 pipeline 작성
$ vi config/test.conf
----------------------
input { 
    tcp {
    	port => 9900
    }
} 

# filter 점진적으로 추가하며 테스트
filter {
#########내용########
}

output { 
    stdout { } 
}
---------------------

# logstash 실행
$ bin/logstash -f config/test.conf

# 새로운 터미널 열어서 netcat명령으로 data 보내기
$ head -n 5 log/test.log | nc localhost 9900
# 위 명령은 [ log/test.log 파일의 처음부터 5 lines을 읽어서 localhost ip주소의 9900포트로 전송 ] 입니다.
# logstash 실행 중인 터미널에서 stdout이 정확하게 출력되는지 확인하며 테스트
```



```bash
# netcat 설치
$ sudo apt-get install netcat

# netcat으로 포트 열림 확인
$ nc -v -z localhost 9200
--------------------------
Connection to localhost 9200 port [tcp/*] succeeded!
```









