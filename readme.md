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

> Docker Postgres

- 설치

``
--postrgresql docker 설치
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