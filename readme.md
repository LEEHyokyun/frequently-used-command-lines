> jar 파일 별도 설치할 필요없이 IntelliJ 내장 jar 파일 사용(환경변수 추가)

- 환경변수

`
$env:PATH = "C:\gradle-9.1.0\bin;C:\Program Files\JetBrains\IntelliJ IDEA 2025.1.2\jbr\bin;$env:PATH"
`

> MSA 마이크로서비스 실행 : 동일한 서비스 이중 실행

- 다른 포트로 이중 실행(port)

``
java -jar ./build/libs/cloud-native-msa-user-1.jar --server.port=60001
``

- 다른 포트로 이중 실행(VM Options)

``
JVM Options
-Dserver.port=600011
``

- hostOS의 포트 다르게 설정(동시 실행 가능)

``
docker run --name cloud-native-msa-order-mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3307:3306 mysql:8.0.42
``
</br>
``
docker run --name cloud-native-msa-product-mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3308:3306 mysql:8.0.42
``

> Docker Volume 확인

- docker volume list

``
docker volume ls
``

- docker volume 세부내역 조회(host 등)

``
docker volume inspect jenkins-tomcat-infra_jenkins_home
``
</br>
``
docker volume inspect jenkins-tomcat-infra_tomcat_webapps
``

> Docker Postgres

- 설치

``
docker run --name postgres-batch -e POSTGRES_USER=root -e POSTGRES_PASSWORD=1234 -p 5432:5432 -d postgres:latest
``

- bash 환경에서 권한 정보 확인

``
cat /var/lib/postgresql/data/pg_hba.conf
``

- Volume Mount 정보 확인 (**경로 확인 : (/data(15.5) 혹은 /postgresql(latest 18+)**)
  - docker 컨테이너 삭제 및 재생성 시 Volumne 중복을 확인하고, 필요 시 볼륨 삭제 가능

``
docker inspect spring-batch-postgresql | grep -A5 Mounts
``
</br>
``
docker inspect spring-security-postgresql | grep -A5 Mounts
``


- 유의사항

``
권한 관련 환경변수 설정 확인 : 15.5/latest 버전은 환경변수가 달라짐
``
</br>
``
echo $PGDATA
``

``
hba_pg.conf에서 권한정보를 md5로 변경,
외부 localhost(tcp) 접속 시 비밀번호를 입력하도록 설정 필요
``

- bash 접속(인증이 필요하지 않은 경우/인증이 필요한 경우)
  - 기본 DB = postgres

``
docker exec -it spring-security-postgresql bash
``
</br>
``
psql -U postgres
``
</br>
``
psql -h localhost -U postgres / password = root
``

> Docker MySQL

- 설치

``
docker run --name cloud-native-rest-api -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 mysql:8.0.42
``

- bash 접속

``
docker exec -it cloud-native-rest-api-mysql bash
``
</br>
``
mysql -u root -p
``

- db 생성 및 접속

``
create database cloudNativeRestApi;
``
</br>
``
use board;
``

> Docker RabbitMQ

- 설치

``
docker run -d --name cloud-native-msa-rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management
``
</br>

- 기본 계정

``
ID: guest
PW : guest
``

> Docker Redis

- 설치

``
docker run -d --name cloud-native-msa-redis -o 6379:6379 redis:7.4
``

> Docker Kafka

- 설치

``
docker run -d --name cloud-native-msa-kafka -p 9092:9092 apache/kafka:3.8.0
``

- sh 실행

``
docker exec --workdir /opt/kafka/bin/ -it cloud-native-msa-kafka sh
``

- kafka 내부 토픽 생성(*토픽 명명 시 내부 도메인 : 이벤트 행위)
 
``
./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic order-dbsync --re
plication-factor 1 --partitions 3
``

- kafka 내부 토픽 확인

``
./kafka-topics.sh --bootstrap-server localhost:9092 --list
``

- kafka 토픽 세부 정보 확인

``
./kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic order-dbsync
``

- kafka 메시지 발행 테스트

``
./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic order-dbsync
``

- kafka 메시지 소비 테스트

``
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic order-dbsync --from-beginning
``

> 컨테이너 연결을 위한 네트워크 생성

- 네트워크 생성(기본 Bridge Driver)

``
docker network create my-bridge-network
``

> 컨테이너 환경 분석 및 확인

- 도커 컨테이너 환경 분석 및 확인

``
docker inspect cloud-native-msa-kafka
``

> Docker Compose

- 도커 compose **(*-f : docker compose.yaml 파일이름, -p : docker ps에 등록할 프로젝트이름)**

``
docker compose -f docker-compose-order-dbsync-kafka-connect.yml -p order-dbsync-kafka-connect up -d
``

- 도커 compose 실행 후 컨테이너 내부 로그 출력

``
docker logs order-dbsync-kafka-connect
``

- compose 중지

``
docker compose -p order-dbsync-infra stop
``

- compose 재실행

``
docker compose -p order-dbsync-infra start
``

> Docker Zipkin

- Zikpin 설치

``
docker run -d --name cloud-native-msa-zipkin -p 9411:9411 openzipkin/zipkin
``

> Docker Grafana

- grafana 도커 설치

``
docker run -d -p 3000:3000 --name=cloud-native-msa-grafana grafana/grafana
``

> Docker Prometheus

- prometheus 도커 설치

``
docker run -d -p 9090:9090 --name=cloud-native-msa-prometheus prom/prometheus
``

> Docker Monitoring System

- prometheus + grafana

``
docker compose -f docker-compose-prometheus-grafana-monitoring-infra.yml -p cloud-native-prometheus-grafana-monitoring-infra up -d
``

> Docker Build & Image Push

- build

``
./gradlew build
``

- jar 실행

``
java -jar build/libs/cloud-native-msa-config-server-1.jar
``

- docker build, push

``
docker build -t leehyokyun/cloud-native-msa-config-server . 
``
</br>
``
docker build -t leehyokyun/cloud-native-msa-config-server:1.0 .
``
</br>
``
docker push leehyokyun/cloud-native-msa-config-server:1.0
``
</br>

- 도커 이미지 실행 실패 시 로그 확인

``
docker logs user-service
``

> Docker Jenkins

- Local 환경에서 Volume 별도 지정 없이 테스트를 진행하기 위한 환경 구성
  - 8080 : jenkins master 실행 및 접속을 위한 포트
  - 50000 : jenkins job 실행을 위한 agent 포트

``
docker run -p 8080:8080 -p 50000:50000 --name cloud-native-cicd-jenkins --restart=on-failure jenkins/jenkins:lts-jdk21
``

- deploy까지 고려하여, 외부 container 환경의 tomcat 인스턴스에 배포(복사)할 경우

``
docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock --name cloud-native-cicd-jenkins jenkins/jenkins:lts-jdk21
``

- docker 컨테이너 환경에서 jenkins 내부 환경으로 접속

``
docker exec -it cloud-native-cicd-jenkins bash
``

- 컨테이너 환경 실행 시 비밀번호 확인

``
docker logs cloud-native-cicd-jenkins
``

- 내부 환경에서의 jdk 정상 인지 여부 확인

``
java -version
echo $JAVA_HOME
``

- workspace(*build/파일 archive 등 확인)

``
/var/jenkins_home/workspace/cloud-native-cicd-jenkins-project-1
``

- build 확인(*jar/war)

``
/var/jenkins_home/workspace/cloud-native-cicd-jenkins-todeploy-1/build/libs
``

> Docker Tomcat

- jenkins와 tomcat의 inner port는 동일, 동시 실행 시 host port는 다르게 설정.

``
docker run -d -p 8081:8080 --name cloud-native-cicd-tomcat tomcat:9.0
``

- manager 권한 계정 추가를 위한 과정

- tomcat-user.xml 파일 확인 

``
cd /usr/local/tomcat/conf
``

- 계정 추가

``
sed -i '/<\/tomcat-users>/i\<role rolename="manager-gui"/>\n<user username="admin" password="admin" roles="manager-gui"/>' tomcat-users.xml
``

- 변경 내용 확인

``
cat tomcat-users.xml
``

- Manager UI 설치(tar(binary) 압축 형태로 다운로드 및 압축 풀기, manager UI 환경 구성)

``
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.80/bin/apache-tomcat-9.0.80.tar.gz
``
</br>
``
tar -xzf apache-tomcat-9.0.80.tar.gz
``
</br>
``
cp -r apache-tomcat-9.0.80/webapps/manager .
``
</br>
``
cp -r apache-tomcat-9.0.80/webapps/host-manager .
``

- context.xml를 통한 외부 접근 권한 수정

``
vi /usr/local/tomcat/webapps/manager/META-INF/context.xml
``

> Docker Network

- Network 생성

``
docker network create cicd-network
``

- Network 연결(*localhost를 통한 내부적 NAT이 아닌 컨테이너명 기반의 통신)

``
docker network connect cicd-network jenkins \n
docker network connect cicd-network ansible \n
docker network connect cicd-network java-app
``

> ngrok

- 현재 실행 중인 application의 포트를 외부에서 접근할 수 있는 uri로 변경
- web hook의 경우 git과 jenkins의 연결이 이루어져야 하므로, jenkins 서버 대상으로 구성 필요.

``
ngrok http 8080 (*Jenkins)
``

> etc

- 도커 명령어 실행을 위한 패키지(최신 라이브러리) 업데이트 및 docker 라이브러리 추가

``
apt-get update
``
``
apt-get install -y docker.io
``

- SSH 통신 환경 구축(공개키 생성 및 target server에 공개키 배포)

``
-- ./ssh 디렉토리에 생성 및 저장
ssh-keygen
``

``
-- 공개키 배포(이것 또한 ./ssh 디렉토리에 배포됨)
ssh-copy-id root@cloud-native-cicd-ansible-server
``

※ target server에 ssh 서버가 설치되어있지 않을 경우, SSH 서버 설치 혹은 docker 기반의 통신을 진행.
