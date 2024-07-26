# WooriFISA Week3 미니프로젝트 : ELK Stack 구축을 통한 Titanic 데이터 시각화
## :notebook: 수행 과제
- MySQL + ELK 연동
- Titanic 데이터 분석 및 시각화

<br/>

## :raising_hand: 팀원
|<img src="https://github.com/leesj000603.png" width="80">|<img src="https://github.com/been980804.png" width="80">|<img src="https://github.com/cshharry.png" width="80">|<img src="https://github.com/yyyeun.png" width="80">|
|:---:|:---:|:---:|:---:|
|[이승준](https://github.com/leesj000603)|[이현빈](https://github.com/been980804)|[조성현](https://github.com/cshharry)|[허예은](https://github.com/yyyeun)|

<br/>

## 실습환경
:penguin: Ubuntu server 22.04.4 LTS
:dolphin: MySQL 8.0.37 
:book: elk stack 7.11.1

## :floppy_disk: MySQL + ELK
### MySQL DB-Connector 설치
```bash
# 다운로드
$ wget 'https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.18.tar.gz'

# 압축 해제
$ tar -xvf ./mysql-connector-java-8.0.18.tar.gz

# logstash bin 디렉터리로 jar 파일 이전
$ sudo cp ./mysql-connector-java-8.0.18/mysql-connector-java-8.0.18.jar /usr/share/logstash/bin

# 압축 해제한 디렉터리 삭제
$ rm -rf ./mysql-connector-java-8.0.18*
```

### Logstash 설정
```bash
# logstash의 conf.d 디렉터리로 이동해 mysql_logstash config 파일 생성
$ cd /etc/logstash/conf.d
$ sudo touch mysql_logstash.conf
```

```bash
input {
  jdbc {
    jdbc_driver_library => "/usr/share/logstash/bin/mysql-connector-java-8.0.18.jar"
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/(DB명)?serverTimezone=Asia/Seoul"
    jdbc_user => "(유저)"
    jdbc_password => "(패스워드)"
    statement => "select * from (Table명)"
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "(index명)"
  }
}
```

### Logstash 실행
```bash
$ sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/mysql_logstash.conf
```

<br/>

## :bar_chart: 시각화

[시각화 아이디어 회의록](https://flower-polyanthus-3b1.notion.site/2024-07-25-be9bf47d5ae64f7885795db54d581d04?pvs=4)

<br/>

## :hammer: 트러블슈팅
____________________________________________________________________________
### Kibana 대시보드 공유
- VirtualBox 네트워크 포트포워딩 수정 : localhost를 호스트 OS의 IP로 지정
____________________________________________________________________________
### JDBC_driver 읽기 권한
```
Error: unable to load /home/username/lib/mysql-connector-java-8.0.18.jar from :jdbc_driver_library, 
file not readable (please check user and group permissions for the path)
```

logstash가 mysql-connector를 읽지 못하는 에러

#### 해결
1) 모든 유저에대한 읽기 권한 부여
```
chmod 644 /home/username/lib/mysql-connector-java-8.0.18.jar
```
또는

2) 파일 소유권 변경
logstash라는 소유자와 그룹에게 해당 파일에 대한 소유권 부여
```
sudo chown logstash:logstash /home/username/lib/mysql-connector-java-8.0.18.jar
```

____________________________________________________________________________
### Timezone error

```
Error: Java::JavaSql::SQLException: The server time zone value 'KST' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```
MySQL 서버에서 설정된 시간대가 JDBC 드라이버에서 인식되지 않는 문제

#### 해결

logstash conf 파일의 jdbc_connection_string 설정에 serverTimezone 설정 추가
```
jdbc_connection_string = "jdbc:mysql://[ip]:[port]/[데이터베이스명]?serverTimezone=Asia/Seoul";
```

____________________________________________________________________________
### conf파일의 filter 충돌

serverTimezone 설정 후 Elasticsearch에 index가 생성

그러나 전혀 관계 없는 컬럼이 생성됨

sudo systemctl enable logstash
설정으로 인해 자동 실행될 때
conf파일을 읽어들이는 과정에서 conf파일의 filter끼리 충돌

#### 해결
1. logstash 자동 시작 해제 및 conf파일 지정하여 logstash 가동

자동시작 해제
```
sudo systemctl disable logstash
```

conf파일을 지정해서 실행
```
sudo /usr/share/logstash/bin/logstash -f /path/to/your/logstash.conf
```

또는

2. logstash.yml 의 path.config 속성 지정
```
path.config : /etc/logstash/single_conf
```


또는

3. pipelines.yml 설정

 logstash 6점대 버전 이상부터 제공하는 pipelines.yml을 통해 파이프라인을 분리하고 각 파이프라인에 사용할 conf파일을 지정

 ```
 - pipeline.id: pipeline1
  path.config: "/etc/logstash/conf.d/pipeline1.conf"

- pipeline.id: pipeline2
  path.config: "/etc/logstash/conf.d/pipeline2.conf"
```