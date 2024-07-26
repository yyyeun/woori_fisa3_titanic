# WooriFISA Week3 미니프로젝트 : ELK Stack 구축을 통한 Titanic 데이터 시각화

<details>
  <summary>목차</summary>  
  
  - [수행 과제](#notebook-수행-과제)
  - [팀원](#raising_hand-팀원)
  - [실습환경](#실습환경)
  - [MySQL + ELK](#floppy_disk-mysql--elk)
    - [MySQL DB-Connector 설치](#mysql-db-connector-설치)
    - [Logstash 설정](#logstash-설정)
    - [Logstash 실행](#logstash-실행)
  - [시각화](#bar_chart-시각화)
  - [트러블슈팅](#hammer-트러블슈팅)
    - [Kibana 대시보드 공유](#kibana-대시보드-공유)
    - [JDBC_driver 읽기 권한](#jdbc_driver-읽기-권한)
    - [Timezone error](#timezone-error)
    - [conf파일의 filter 충돌](#conf파일의-filter-충돌)
      
</details>



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

![alt text](image.png)

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
    jdb고
