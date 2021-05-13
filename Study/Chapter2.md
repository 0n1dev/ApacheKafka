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

### kafka-topics.sh

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
- 다이나믹 토픽 옵션인 `retention.ms`를 86400000ms로 변경

### kafka-console-producer.sh

> 토픽에 메세지를 넣을 때 사용

**토픽에 메세지 전송**

> --bootstrap-server <KAFKA_HOST>:<PORT> --topic hello-kafka

- 메세지는 기본적으로 key, value로 이루어짐
- 키 없이 메세지 값만 보내면 키는 null로 기본 설정

**토픽에 키를 가지는 메시지 전송**

> --bootstrap-server <KAFKA_HOST>:<PORT> --topic hello-kafka --property "parse.key=true" --property "key.separator=:"

- parse.key를 true로 두면 레코드를 전송할 때 메세지 키를 추가
- `key.separator`로 구분자 설정 (지정 구분자를 넣지 않으면 KafkaException 발생)
- 오프셋-메시지 키-메시지 값으로 묶인 레코드가 토픽의 파티션에 전송

-> 메시지 키가 동일한 파티션으로 전송 (파티셔너가 커스터마이징 된 경우 다르게 동작)

### kafka-console-consumer.sh

> 토픽으로 전송한 메세지를 확인 가능

- --from-beginning 옵션을 주면 토픽에 저장된 가장 처음 데이터부터 출력

**토픽 메세지 조회**

> --bootstrap-server <KAFKA_HOST>:<PORT> --topic hello-kafka --from-beginning

**키값을 포함한 토픽 메세지 조회**

> --bootstrap-server <KAFKA_HOST>:<PORT> --topic hello-kafka --property print.key=true --property key.separator="-" --group hello-group --from-beginning

- `print.key`를 true로 설정하면 키 확인 가능
- `key.separator`를 통해 구분자 지정
- `--group` 옵션을 통해 신규 컨슈머 그룹 생성 (컨슈머 그룹을 통해 가져간 토픽의 메세지에 대해 커밋)

-> 커밋이란 컨슈머가 특정 레코드까지 처리를 완료했다고 레코드의 오프셋 번호를 카프카 브로커에 저장 (커밋 정보는 __consumer_offsets 이름의 내부 토픽에 저장)

### kafka-consumer-groups.sh

> 따로 명령어를 통해 생성이 아닌 컨슈머 동작시 컨슈머 그룹의 이름을 지정하면 생성

**컨슈머 그룹의 리스트 조회**

> --bootstrap-server <KAFKA_HOST>:<PORT> --list

**컨슈머 그룹에 대한 상세 내용 조회**

> --bootstrap-server <KAFKA_HOST>:<PORT> --group hello-group --describe

### kafka-verifiable-producer, consumer.sh

> kafka-verifiable로 시작하는 2개의 스크립트를 사용하면 String 타입 메시지 값을 코드 없이 주고받을 수 있음

**producer.sh를 통해 메세지 전송**

> kafka-verifiable-producer.sh --bootstrap-server <KAFKA_HOST>:<PORT> --max-messages 10 --topic verify-test

**consumer.sh를 통해 메세지 조회**

> kafka-verifiable-consumer.sh --bootstrap-server <KAFKA_HOST>:<PORT> --topic verify-test --group-id test-group

### kafka-delete-records.sh

> 이미 적재된 토픽의 데이터 삭제 (가장 오래된 데이터부터 특정 시점의 오프셋까지 삭제 가능)

**토픽의 데이터 삭제**

> --bootstrap-server <KAFKA_HOST>:<PORT> --offset-json-file delete-topic.json

- `--offset-json-file` 삭제 토픽, 파티션, 오프셋에 대한 정보를 담은 delete-topic.json을 읽어서 삭제

-> 특정 데이터만 삭제 불가능