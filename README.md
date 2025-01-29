## 목적
리눅스서버 에서 ELK Stack을 설치하여 가용 가능하도록 인프라를 구성하는 것을 연습 하기위해 작성되었습니다.

## 1. VM 준비
Virtual box 내에 우분투가 설치된 가상머신 2대를 준비한다.

두 리소스 간 연결을 위해 각각의 가상 머신에서 IP 정보를 확인한다.

``` bash
ip addr
```

| vm_es | vm_kb |
| ----- | ----- |
| 10.0.2.15 | 10.0.2.4 |

## 2. NAT network 설정
가상머신간 통신과 외부 통신을 지원하는 NAT network을 설정한다.

### 2-1 NAT network 생성
### 2-2 DHCP 해제
로컬에서 고정적인 ip로 세션 연결과 ElasticSearch 와 Kibana에서 설정할 ip 값을 고정적으로 할당하기 위해 DHCP 옵션을 해제한다.

### 2-3 포트포워딩 규칙 추가
| Host| Guest |
| --- | ----- |
| localhost:22 | 10.0.2.15:22 |
| localhost:23 | 10.0.2.15:22 |
| localhost:9200 | 10.0.2.15:9200 |
| localhost:5601 | 10.0.2.15:5601 |

## 3. VM_ES에 ElasticSearch 설치

### 3-1. 설치과정

#### .deb 다운로드

``` bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-amd64.deb
```
#### 패키지 다운로드

``` bash
$ sudo dpkg -i elasticsearch-7.11.1-amd64.deb
```
``` bash
# 의존성 발생 시 다음을 실행
$ sudo apt --fix-broken install -y
```

#### 서비스 활성화 및 시작, 상태 확인

``` bash
$ sudo systemctl enable elasticsearch
```

``` bash
$ sudo systemctl start elasticsearch
```

``` bash
$ sudo systemctl status elasticsearch
```

### 3-2. elasticsearch.yml 수정

``` bash
$ sudo vi /etc/elasticsearch/elasticsearch.yml
```

다음 부분을 찾아 수정하거나 추가한다.
``` yml
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
```

### 3-3. elasticsearch 서비스 재시작

``` bash
$ sudo systemctl restart elasticsearch
```

### 3-4. 접속 확인
#### VM_ES 내에서 확인

``` bash
$ curl -X GET "http//10.0.2.15:9200"
```
출력 :
``` json
{
  "name" : "myserver1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "dQuF2aWfS0aXnJBFXbTlKg",
  "version" : {
    "number" : "7.11.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "ff17057114c2199c9c1bbecc727003a907c0db7a",
    "build_date" : "2021-02-15T13:44:09.394032Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

#### 로컬 브라우저에서 확인
`` http://localhost:9200 ``으로  접속 후 elasticsearch 상태가 정상적으로 응답하는지 확인한다.


## 4. VM_KB에 Kibana 설치
### 4-1. 설치과정
#### .deb 다운로드

``` bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.11.1-amd64.deb
```

#### 패키지 설치

``` bash
sudo dpkg -i kibana-7.11.1-amd64.deb
```

``` bash
# 종속성 문제 발생 시
sudo apt-get install -f
```


### 4-2. Kibana.yml 수정

```  yml
# Elasticsearch 호스트 설정
elasticsearch.hosts: ["http://10.0.2.15:9200"]

# Kibana의 서버 호스트(기본: localhost, 모든 IP에서 접근하려면 0.0.0.0)
server.host: "0.0.0.0"
```

만약 elasticsearch 8이상일 경우 xpack 보안옵션이 포함되어 있어 username과  password 를 지정해줘야 한다.

```
elasticsearch.username: "<username>"
elasticsearch.password: "<password>"
```

### 4-3. kibana 서비스 재시작

``` bash
sudo systemctl restart kibana
```

### 4-4. 접속 확인
#### VM_ES와 네트워크 연동 확인

``` bash
curl -X GET "10.0.2.15:9200"
```

요청 응답이 올 경우 kibana가 설치된 vm에서 elasticsearch vm으로 연동이 가능한 네트워크 상태를 의미한다.

#### 로컬 브라우저에서 확인
`` http://localhost:5601 `` 접속 시 kibana 대쉬보드에 접속이 되면 성공이다.

