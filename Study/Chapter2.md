# 카프카 빠르게 시작해보기

### 도커 이미지

```yml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

> docker-copose up -d

## 카프카 제공 커맨드 라인 툴

> 브로커 운영에 필요한 다양한 명령어를 내릴 수 있음

### Kafka-topics.sh

> 토픽과 관련된 명령 실행

-> 카프카 컨슈머나 프로듀서가 브로커에 생성되지 않은 토픽에 대해 데이터를 요청할 때 자동으로 생성됨 (명시적으로 생성하는게 유지 보수하기 좋음)

**토픽 생성**

> bin/kafka-topics.sh --create --bootstrap-server <KAFKA_HOST>:<PORT> --topic hello-kafka

- `--create` 토픽을 생성하는 명령어라는 것을 명시
- `--bootstrap-server` 토픽을 생성할 카프카 클러스터를 구성하는 브로커의 IP와 port 작성
- `--topic` 토픽의 이름 작성

> bin/kafka-topics.sh --create --bootstrap-server <KAFKA_HOST>:<PORT> --partitions 3 --replication-factor 1 --config retention.ms=172800000 --topic hello-kafka2

- `--partions` 파티션의 개수 최소 1개 옵션을 미설정시 config/server.properties에 있는 num.partitions 옵션을 참고하여 생성
- `replication-factor` 토픽의 파티션을 복제할 복제 개수 1은 복제하지 않는다는 의미 최대 개수는 카프카 클러스터의 브로커 개수
- `--config` kafka-topics.sh 명령에 포함되지 않은 추가적인 설정 가능. 예제에서 작성한건 데이터 유지시간을 밀리세컨드로 표기

**토픽 리스트 조회**

> bin/kafka-topics.sh --bootstrap-server <KAFKA_HOST>:<PORT> --list

**토픽 상세 조회** (여기부터 bin/kafka-topics.sh는 생략하고 작성예정)

> --bootstrap-server <KAFKA_HOST>:<PORT> --describe --topic hello-kafka

**토픽 옵션 수정**

- 파티션 개수 변경 kafka.topics.sh 사용
- 토픽 삭제 정책 리테션 기간 변경은 kafka-configs.sh 사용

> --bootstrap-server <KAFKA_HOST>:<PORT> --topic hello-kafka --alter --partitions 4

- `--alter` 옵션과 `--partitions` 옵션을 함께 사용하면 파티션 개수 변경

> bin/kafka-configs.sh --bootstrap-server <KAFKA_HOST>:<PORT> --entity-type topics --entity-name hello-kafka --alter --add-config retention.ms=86400000

- `--alter`, `--add-config` 두개 옵션을 같이 사용하면 설정값이 이미 존재하면 수정 존재하지 않으면 신규로 추가