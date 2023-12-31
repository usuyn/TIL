> [아파치 카프카 입문](https://www.inflearn.com/course/아파치-카프카-입문) 강의를 보고 정리합니다.

## 목차

1. [아파치 카프카 개요 및 설명](#아파치-카프카-개요-및-설명)
   - [Before Kafka](#1-before-kafka)
   - [Kafka 주변 생태계](#2-kafka-주변-생태계)
   - [결론](#3-결론)
2. [Topic이란?](#topic이란)
   - [Topic](#1-topic)
   - [Topic 내부, Partition](#2-topic-내부-partition)
   - [Partition이 2개 이상인 경우](#3-partition이-2개-이상인-경우)
   - [Partition을 늘리는 것](#4-partition을-늘리는-것)
   - [Partition의 Record가 삭제되는 시점](#5-partition의-record가-삭제되는-시점)
3. [Broker, Replication, In-Sync Replica](#broker-replication-in-sync-replica)
   - [Broker](#kafka-broker)
   - [Replication](#kafka-replication)
   - [ISR](#kafka-isrin-sync-replica)
   - [Why replicate?](#why-replicate)
   - [Replication & ack](#replication--ack)
   - [Replication count](#replication-count)
4. [Partitioner란?](#partitioner란)
   - [메시지 키의 유무](#메시지-키의-유무)
   - [Custom Partitioner](#custom-partitioner)
5. [Consumer Lag이란?](#consumer-lag이란)
6. [Consumer Lag Monitoring Application, Kafka Burrow](#consumer-lag-monitoring-application-kafka-burrow)
   - [Burrow의 특징](#1-burrow의-특징)
   - [결론](#2-결론)
   - [참고 링크](#3-참고-링크)
7. [Message Broker, Event Broker(Kafka, RabbitMQ, Redis Queue의 차이점)](#message-broker-event-brokerkafka-rabbitmq-redis-queue의-차이점)
   - [Message Broker](#1-message-broker)
   - [Event Broker](#2-event-broker)
8. [AWS에 Kafka cluster 설치, 실행](#aws에-kafka-cluster-설치-실행)
   - [EC2 인스턴스 발급](#ec2-인스턴스-발급)
   - [security group inbound 규칙 설정](#security-group-inbound-규칙-설정)
   - [/etc/hosts 설정](#etchosts-설정)
   - [Zookeeper 설치](#zookeeper-설치)
   - [Zookeeper 실행](#zookeeper-실행)
   - [Kafka 설치](#kafka-설치)
   - [Kafka 실행](#kafka-실행)
   - [console-producer, consumer 테스트](#console-producer-consumer-테스트)
9. [Kafka Producer Application](#kafka-producer-application)
   - [Kafka Producer의 역할](#kafka-producer의-역할)
   - [Producer Application 개발](#producer-application-개발)
     - [Java Properties 객체를 통한 Producer 설정 및 생성](#java-properties-객체를-통한-producer-설정-및-생성)
     - [ProducerRecord class 사용한 전송 객체 생성](#producerrecord-class-사용한-전송-객체-생성)
     - [전송 및 종료](#전송-및-종료)
10. [Kafka Consumer Application](#kafka-consumer-application)
    - [Kafka Consumer의 역할](#kafka-consumer의-역할)
    - [Consumer Application 개발](#consumer-application-개발)
      - [Java Properties 객체를 통한 Consumer 설정 및 생성](#java-properties-객체를-통한-consumer-설정-및-생성)
      - [대상 topic 지정](#대상-topic-지정)
      - [polling loop](#polling-loop)
    - [Producer에서 Consumer로 Message 전달](#producer에서-consumer로-message-전달)
    - [Multiple Consumer](#multiple-consumer)
    - [Different Groups](#different-groups)
11. [Kafka Streams Application](#kafka-streams-application)
    - [Kafka Streams의 장점](#kafka-streams의-장점)
    - [Streams DSL이 제공하는 이벤트 기반 메서드](#streams-dsl이-제공하는-이벤트-기반-메서드)

## 아파치 카프카 개요 및 설명

### **1. Before Kafka**

**데이터를 전송하는 Source Application**과 **데이터를 받는 Target Application**이 존재하며, 초기에는 단방향 통신을 사용했다.

시간이 흐름에 따라 Source와 Target application이 많아지면서 데이터 전송 라인 또한 복잡해졌다.

**데이터 전송 라인이 복잡**해지면 **배포, 장애에 대응하기 어려워**진다. 또한 데이터를 전송할 때 **프로토콜, 포맷의 파편화**가 심해지며, 데이터 포맷 변경사항이 있을 때 **유지보수도 어려워**진다.

이러한 **문제들을 해결**하기 위해 LinkedIn에서 **Apache Kafka를 개발**했고 현재는 오픈소스로 제공되고 있다.

---

### **2. Kafka 주변 생태계**

kafka는 **source와 target application의 커플링을 약하게** 하기 위해 만들어졌다.

**Source application은 kafka에 데이터를 전송**하고,  
**Target application은 kafka에서 데이터를 가져온다.**

Source application에서 **전송할 수 있는 데이터의 포맷은 거의 제한이 없다.** json, tsc, avro 등을 모두 지원한다.

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/da548445-f12e-4554-87fb-35cbfec2cec8">  
  
Kafka에는 **각종 데이터를 담는 Topic**이라는 개념이 존재한다. 쉽게 말하면 **Queue**인 것이다.

**Queue에 데이터를 넣는 Kafka Producer**와  
**Queue에서 데이터를 가져가는 Kafka Consumer**로 나누어진다.

**producer와 consumer는 라이브러리**로 되어 있어 application에서 구현 가능하다.

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/dd5b1861-3ded-4762-99c6-c9de8f9348b2">

---

### **3. 결론**

Kafka는 **아주 유연한 Queue** 역할을 하는 것이라고 할 수 있다.

Kafka는 데이터 흐름에 있어 Fault Tolerant 즉, **고가용성**으로 서버에 이슈가 생기거나 갑자기 전원이 내려가도 **데이터 손실 없이 복구가 가능**하다. 또한 낮은 지연(latency), 높은 처리량(throughput)을 통해 **효율적으로 많은 데이터 처리**가 가능하다.

## Topic이란?

### **1. Topic**

데이터가 들어가는 공간을 topic이라고 부른다.

kafka topic은 일반적인 AMQP(Advanced Message Queing Protocol)와는 다르게 동작한다.

&#62; AMQP(Advanced Message Queing Protocol) : 메시지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜, MQ의 오픈 소스에 기반한 표준 프로토콜을 의미한다. 기존 MQ들의 약점을 보완하기 위해 등장했다.

kafka에서는 topic을 여러개 생성할 수 있으며, DB의 테이블이나 파일 시스템의 폴더와 유사한 성질을 가진다.

topic은 이름을 가질 수 있다. 어떤 데이터를 담는지, 목적에 따라 명확하게 명시하면 추후 쉽게 유지보수가 가능하다.

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/0178463e-c98e-41b6-b840-550334dae3e0">

---

### **2. Topic 내부, Partition**

하나의 topic은 여러개의 partition으로 구성될 수 있다.

partition 번호는 0번부터 시작하며, queue와 같이 끝에서부터 데이터가 차곡차곡 쌓인다.

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e558b3f4-2928-4360-a480-995f692cff8c">

한 topic에 consumer가 붙게 되면 가장 오래된 데이터부터 순서대로 가져가며, 더이상 데이터가 없으면 또 다른 데이터가 들어올 때까지 기다린다.

consumer가 topic 내부 partition에서 데이터를 가져가도 데이터는 삭제되지 않는다. 따라서 새로운 consumer가 붙었을 때 다시 0번 데이터부터 가져갈 수 있다.

다만 consumer 그룹이 달라야 하며, auto.offset.reset = earliest로 세팅되어 있어야 한다.

&#62; offset : consumer가 topic의 데이터를 어디까지 읽었는지 저장값

&#62; auto.offset.reset 옵션

```java
// consumer가 topic 구독 후 partition에 처음 들어오는 데이터부터
auto.offset.reset = latest

// 가장 처음 offset부터 즉, 가장 오래된 데이터
auto.offset.reset = earliest

// offset 찾지 못하면 exception 발생
auto.offset.reset = none
```

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8a36ff31-0c63-47a5-b0c7-8984ca139edc">

데이터가 삭제되지 않기 때문에 동일한 데이터에 대해 2번 처리할 수 있는데 이는 kafka를 사용하는 아주 중요한 이유기도 하다.

클릭 로그를 분석하고 시각화하기 위해 ES(Elasticsearch)에 저장하기도 하고, 클릭 로그를 백업하기 위해 Hadoop에 저장할 수도 있다.

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/6bb3dfa0-3dda-4956-91ee-565971e51157">

---

### **3. Partition이 2개 이상인 경우**

producer는 데이터를 보낼 때 키를 지정할 수 있다.

producer가 새로운 데이터를 보내면 아래 규칙에 따라 partition이 지정된다.

- key가 null이고, 기본 partitioner 사용할 경우  
  $\rightarrow$ RR(Round-robin)로 할당

- key가 있고, 기본 partitioner 사용할 경우  
  $\rightarrow$ key의 hash값을 구하고 특정 partition에 할당한다.

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ccecd73e-5c9f-4e35-b437-fad6d9ac99f7">

---

### **4. Partition을 늘리는 것**

partition을 늘리는 것은 가능하나 다시 줄일 수는 없다.

그렇다면 왜 partition을 늘리는 것일까?

바로 데이터 처리를 분산시키기 위해서이다. partition을 늘리면 consumer를 늘려서 데이터 처리를 분산시킨다.

---

### **5. Partition의 Record가 삭제되는 시점**

삭제되는 타이밍은 옵션에 따라 다르며, 아래 방법으로 record가 저장되는 최대 시간과 길이를 지정할 수 있다.

```java
// 최대 record 보존 시간
log.retention.ms

// 최대 record 보존 크기(byte)
log.retention.byte
```

위 방법을 통해 일정한 기간 혹은 용량만큼 데이터를 저장할 수 있게 되며, 적절한 시점에 데이터가 삭제되게 설정할 수 있다.

## Broker, Replication, In-Sync Replica

Broker, Replication, ISR(In-Sync Replica)은 kafka 운영에 있어서 아주 중요한 역할을 한다.  
kafka 아키텍처의 핵심인 replication(복제)은 클러스터에서 서버에 장애가 생겼을 때 kafka의 가용성을 보장하는 가장 좋은 방법이다.

### Kafka Broker

kafka broker란 kafka가 설치되어 있는 서버 단위이다.  
보통 3개 이상의 broker를 구성해 사용하는 것을 권장한다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/fab4327c-c107-4f8a-be9c-8ddf72a293ac">

만약 partition이 1개이고 replication이 1인 topic이 존재하며 broker가 3대라면 3대 중 1대에 해당 topic의 정보(데이터)가 저장된다.

### Kafka Replication

replication은 partition의 복제이다.

- **replication : 1** $\rightarrow$ partition 1개만 존재

- **replication : 2** $\rightarrow$ 원본 partition 1개 + 복제본 partition 1개

- **replication : 3** $\rightarrow$ 원본 partition 1개 + 복제본 partition 2개

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/2bdc3ad6-a0fb-43a0-abc3-ae732de030c6">

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/fa6fa585-0372-432d-92da-a0c72eeee6b0">

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/88564b2f-9cbf-4ce8-bf82-b9a388a416fb">

다만 broker 개수에 따라 replication 개수가 제한된다. broker가 3인 경우 replication는 4가 될 수 없다.

원본 partition은 Leader partition이라고 부른다.  
나머지 복제본 partition은 Follower partition이라고 부른다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8ba1d6c6-05eb-4545-8421-fc40b8adc118">

### Kafka ISR(In-Sync Replica)

leader partition과 follower partition들의 그룹이다.  
그룹에 속해있다는 것은 leader의 데이터와 동기화되어 있다는 것을 의미하며 leader에 문제가 생길 시 언제든 그 자리를 대신할 수 있게 된다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/7a197fda-0b1a-4601-990c-5bc7abd4e1d2">

### Why replicate?

partition의 고가용성을 위해 사용된다.

만약 broker가 3개인 kafka에서 replication가 1이고 partition이 1인 topic이 존재한다고 가정한다. 갑자기 broker가 어떠한 이유로 사용할 수 없게 되면 더이상 해당 partition은 복구할 수 없다.  
만약 replication가 2이면 broker 1개가 죽더라도 복제본 즉, follower partition이 존재하므로 복구가 가능하다. 해당 follower partition이 리더 partition 역할을 승계하게 된다.

### Replication & ack

producer가 topic의 partition에 데이터를 전달한다. producer가 topic의 partition에 데이터를 전달할 때 전달받는 주제가 바로 leader partition이다.

producer에는 ack라는 상세 옵션이 있다. ack를 통해 고가용성을 유지할 수 있는데 이 옵션은 partition의 replication와 관련이 있다.

ack는 0, 1, all 옵션 3개 중 한개를 골라 설정할 수 있다.

- **ack : 0** $\rightarrow$ producer는 leader partition에 데이터를 전송하고 응답값을 받지 않는다. leader partition에 데이터가 정상적으로 전송됐는지, 나머지 partition에 정상적으로 복제됐는지 알 수 없고 보장할 수 없다. 속도는 빠르나 데이터 유실 가능성이 있다.

- **ack : 1** $\rightarrow$ producer는 leader partition에 데이터를 전송하고 응답값을 받는다. 하지만 나머지 partition에 정상적으로 복제됐는지 알 수 없다.  
  leader partition이 데이터를 받은 즉시 브로커에 장애가 생기면 나머지 partition에 데이터가 전송되지 못한 상태이므로 데이터 유실 가능성이 있다.

- **ack : all** $\rightarrow$ producer는 leader partition에 데이터를 전송하고 응답값을 받으며 나머지 follower partition에 복제가 잘 이루어졌는지에 대한 응답값도 받는다.  
  데이터 유실 가능성은 없으나 옵션 0, 1에 비해 확인하는 부분이 많기 때문에 속도가 현저히 느리다는 단점이 있다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d887c5f4-8eba-4234-a64b-f7293cc9bf49">

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/214cf259-e3c6-41b1-8893-70c46d64ee22">

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/a2793c5e-912b-4a10-9156-a82654e764c1">

### Replication count

replication이 고가용성을 위해 중요한 역할을 하지만 많을수록 좋은 것은 아니다.  
replication 개수가 많아지면 그만큼 broker의 리소스 사용량도 늘어난다.

따라서 kafka에 들어오는 데이터의 양과 retention date(저장 시간)를 잘 생각해 replication 개수를 정하는 것이 좋다.  
3개 이상의 broker를 사용할 때 replication는 3으로 설정하는 것을 추천한다.

## Partitioner란?

partitioner는 kafka producer의 주요 개념 중 하나이다.  
partitioner를 알면 파티션을 더 효과적으로 쓸 수 있다.

producer가 데이터를 보내면 무조건 partitioner를 통해 broker로 데이터가 전송된다.
partitioner는 데이터를 topic의 어느 파티션에 넣을지 결정하는 역할을 한다. 파티션의 위치는 레코드에 포함된 메시지 키 또는 메시지 값에 따라 결정된다.

<img width="200" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/87ec4653-2adc-43ad-8661-743c0dfb477c">

producer를 사용할 때 partitioner를 따로 설정하지 않으면 UniformStickyPartitioner로 설정되는데 이 partitioner는 메시지 키가 있을 때와 없을 때 다르게 동작한다.

### 메시지 키의 유무

- 메시지 키가 있는 경우

  $\rightarrow$ 메시지 키를 가진 레코드는 partitioner에 의해서 특정한 hash 값이 생성된다. 이 hash 값을 기준으로 어떤 파티션에 들어갈지 결정한다.  
   동일한 메시지 키를 가진 레코드는 동일한 hash 값을 만들어내므로 항상 같은 파티션에 들어가는 것을 보장한다.  
   또한 동일한 파티션에 들어가기 때문에 순서를 지켜서 데이터를 처리할 수 있다는 장점이 있다.

<img width="200" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/f9cc70ef-99cd-44f2-a5e8-1766868f460b">

topic에 파티션이 2개 있다고 가정하면  
partitioner의 hash 로직에 의해 서울은 파티션 0번, 부산은 1번, 울산은 0번으로 들어갈 수 있다.

<img width="200" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/66ce2040-a22f-4e76-92ac-e19bedb1bb7b">

예로 서울의 온도를 기록하는 레코드를 파티션에 넣는다고 가정하면  
메시지 키에 "서울"이라는 String 값을 넣고 레코드를 지속적으로 보낸다면 항상 동일한 파티션에 데이터가 순서대로 들어가기 때문에 consumer는 서울이라는 레코드를 순서를 지켜서 데이터를 처리할 수 있다.

파티션 1개 내부에서는 queue처럼 동작하기 때문에 순서를 지킬 수 있게 되는 것이다.

- 메시지 키가 없는 경우

  $\rightarrow$ 메시지 키가 없는 레코드는 round-robin 방식으로 파티션에 들어간다.  
  전통적인 round-robin 방식과는 조금 다르게 동작한다.

UniformStickyPartitioner는 producer에서 배치로 모을 수 있는 최대한의 레코드들을 모아서 파티션으로 데이터를 보내게 된다.  
이렇게 배치 단위로 데이터를 보낼 때 파티션에 round-robin 방식으로 돌아가면서 데이터를 넣게 된다.  
쉽게 말해 메시지 키가 없는 레코드들은 파티션에 적절히 분배되는 방식이다.

### Custom Partitioner

직접 개발한 partitioner도 producer에서 설정할 수 있다.  
kafka에서는 custom partitioner를 만들 수 있도록 partitioner 인터페이스를 제공하고 있다.

partitioner 인터페이스를 사용해 custom partitioner를 만들면 메시지 키 또는 메시지 값 또는 topic 이름에 따라서 어느 파티션에 데이터를 보낼 것인지 정할 수 있다.

- custom partitioner 사용 예시

  $\rightarrow$ VIP 고객을 위해 데이터 처리를 더 빠르게 하는 로직을 생각한다면 partitioner를 통해서 처리량을 늘릴 수 있다.  
  10개의 파티션이 있다고 가정할 때 custom partitioner를 만들어서 8개의 파티션에는 VIP 고객의 데이터를 넣고, 나머지 2개 파티션에는 일반 고객의 데이터를 넣는다.  
  이와 같이 데이터 처리량을 VIP 고객을 위해 몰아주는 형태로 개발 가능하다.

  AMQP 기반 메시징 시스템에서 우선순위 queue를 만드는 것과 비슷하다.

## Consumer Lag이란?

kafka를 운영함에 있어 아주 중요한 모니터링 지표이다.

producer는 topic의 partition에 데이터를 차곡차곡 넣는다. 이 partition에 데이터가 하나씩 들어가게 되면 각 데이터에는 offset이라는 숫자가 붙게 된다.  
partition이 1개일 때 데이터를 넣을 경우 차례대로 숫자가 0부터 매겨진다.

만약 producer가 데이터를 넣어주는 속도가 consumer가 가져가는 속도보다 빠르면  
$\rightarrow$ producer가 넣은 데이터의 offset, consumer가 가져간 데이터의 offset 간 차이가 발생한다.

이것이 바로 consumer lag이다.

<img width="250" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/180b61f1-df97-4d75-b513-a4e9871a8974">
   
lag의 숫자를 통해 해당 topic에 대해 파이프라인으로 연계되어 있는 producer와 consumer 상태 유추가 가능하다. 주로 consumer의 상태를 볼 때 사용한다.

topic에 여러 partition이 존재하면 lag이 여러개 존재할 수 있다.  
consumer 그룹이 1개이고, partition이 2개인 topic에서 데이터를 가져가면 lag 2개가 측정될 수 있다.

이렇게 1개의 topic과 consumer 그룹에 대한 lag이 여러개 존재할 수 있을 때, 그 중 높은 숫자의 lag을 records-lag-max라고 한다.

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d3dbb0e1-5c98-424b-82a4-a023e19d70ae">
   
consumer의 성능이 안 나오거나 비정상적인 동작을 하면 lag이 필연적으로 발생해 주의 깊게 살펴본다.

### 핵심

1. lag은 producer의 offset과 consumer의 offset간의 차이이다.
2. lag은 여러개 존재할 수 있다.

## Consumer Lag Monitoring Application, Kafka Burrow

kafka lag을 모니터링하기 위해 오픈소스인 burrow를 사용한다.

kafka lag은 topic의 가장 최신 offset과 consumer offset간의 차이이다.

kafka client 라이브러리를 사용해 java 또는 scala 같은 언어로 kafka consumer를 구현할 수 있다.  
이때 구현한 kafka consumer 객체를 통해 현재 lag 정보를 가져온다.  
만약 lag을 실시간으로 모니터링 하고 싶다면 데이터를 Elasticsearch, InfluxDB와 같은 저장소에 넣은 뒤 Grafana 대시보드를 통하는 방법을 사용할 수 있다.

consumer 단위에서 lag을 모니터링하는 것은 아주 위험하고 운영 요소가 많이 들어간다.  
그 이유는 consumer 로직단에서 lag을 수집하는 것은 consumer 상태에 dependency가 걸리기 때문이다.

- consumer가 비정상적으로 종료되면 consumer는 lag 정보를 보낼 수 없기 때문에 더이상 lag을 측정할 수 없다.
- consumer가 개발될 때마다 해당 consumer에 lag 정보를 특정 저장소에 저장할 수 있도록 로직을 개발해야 한다. 만약 consumer lag을 수집할 수 없는 consumer라면 모니터링 할 수 없어 운영이 매우 까다로워 진다.

따라서 LinkedIn에서는 apache kafka와 함께 kafka의 consumer lag을 효과적으로 모니터링할 수 있도록 burrow를 배포했다.

---

### 1. Burrow의 특징

burrow는 오픈소스로서 Golang으로 작성되었고 현재 깃허브에 올라와있다.  
consumer lag 모니터링을 도와주는 독립적인 애플리케이션다.

burrow는 3가지 큰 특징을 가지고 있다.

- Multiple Kafka Cluster 지원

kafka cluster를 운영하는 기업이라면 대부분 2개 이상의 kafka cluster를 운영하고 있다. cluster가 여러개더라도 burrow application 1개만 실행해서 연동하면 cluster들에 붙은 consumer lag을 모두 모니터링 할 수 있다.

- Sliding window를 통한 Consumer의 status 확인

sliding window를 통해서 consumer의 status를 ERROR, WARNING, OK로 표현할 수 있도록 했다.

데이터의 양이 일시적으로 많아지면서 consumer offset이 증가되고 있으면 WARNING으로 정의된다.  
데이터의 양이 많아지고 있는데 consumer가 데이터를 가져가지 않으면 ERROR로 정의된다.

- HTTP api 제공

위와 같은 정보들을 burrow가 정의한 HTTP api를 통해 조회할 수 있다.  
범용적으로 사용되는 HTTP를 사용해 다양한 추가 생태계를 구축할 수 있게 된다. HTTP api를 호출해서 response 받은 데이터를 시계열 DB와 같은 곳에 저장하는 application을 만들어 활용할 수도 있다.

---

### 2. 결론

Burrow를 도입한다고 모든 문제가 해결되는 것은 아니다. 다만 kafka 개발자, kafka cluster 운영자가 효과적으로 kafka 관련 애플리케이션을 운영할 때 반드시 필요하며 burrow를 통해 수집된 데이터는 결국 추후 애플리케이션 개발과 운영에 많은 도움이 된다.

---

### 3. 참고 링크

[Burrow github](https://github.com/linkedin/Burrow)  
[Burrow 소개](https://blog.voidmainvoid.net/243)  
[Burrow의 Consumer status 확인 방법](https://blog.voidmainvoid.net/244)  
[Burrow http endpoint 정리](https://blog.voidmainvoid.net/245)

## Message Broker, Event Broker(Kafka, RabbitMQ, Redis Queue의 차이점)

메시징 플랫폼은 2가지 종류로 나누어진다.

1. Message Broker
2. Event Broker

메시지 브로커는 이벤트 브로커 역할을 할 수 없지만, 이벤트 브로커는 메시지 브로커 역할을 할 수 있다.

---

### 1. Message Broker

많은 기업들의 대규모 메시지 기반 미들웨어 아키텍처에서 사용된다. 메시징 플랫폼, 인증 플랫폼, 데이터베이스 등이 미들웨어이다.  
메시지 브로커에 있는 큐에 데이터를 보내고 받는 producer와 consumer를 통해 메시지를 통신하고 네트워크를 맺는 용도로 사용해왔다.

메시지를 받아서 적절히 처리하면 즉시 또는 짧은 시간 내에 삭제되는 구조를 가지고 있다.

---

### 2. Event Broker

이벤트 또는 메시지라고도 불리는 레코드, 장부를 딱 하나만 보관하고 인덱스를 통해 개별 액세스를 관리하며, 업무상 필요한 시간동안 이벤트를 보존할 수 있다.

메시지 브로커는 데이터를 보내고 처리하고 삭제한다.  
이벤트 브로커는 삭제하지 않는다.

이벤트 브로커는 서비스에서 나오는 이벤트를 마치 데이터베이스에 저장하듯이 이벤트 브로커의 큐에 저장한다.
이렇게 저장함으로써 얻는 이점은 아래와 같다.

1. 딱 한번 일어난 이벤트 데이터를 브로커에 저장함으로써 단일 진실 공급원으로 사용 가능
2. 장애 발생했을 때 발생 시점부터 재처리 가능
3. 많은 양의 실시간 스트림 데이터를 효과적으로 처리 가능
4. 다양한 이벤트 기반 마이크로서비스 아키텍처에서 중요한 역할 수행

&#62; 단일 진실 공급원(SSOT; single source of truth) : 정보 시스템 설계 및 이론 중 하나로 정보와 스키마를 오직 하나의 출처에서만 생성, 편집하도록 하는 방법론이다. 단일 출처를 통해 데이터를 생성, 편집, 접근하므로 데이터의 정합성을 지키고 잘못된 데이터 유통을 방지하며 모두가 동일한 데이터를 참고할 수 있다.

메시지 브로커의 예로는 RabbitMQ, Redis Queue가 있으며,  
이벤트 브로커의 예로는 Kafka, AWS Kinesis가 있다.

이벤트 브로커로 클러스터를 구축하면 이벤트 기반 마이크로서비스 아키텍처로 발전하는데 아주 중요한 역할을 할 뿐만 아니라 메시지 브로커로서도 사용할 수 있다.

## AWS에 Kafka cluster 설치, 실행

aws의 ec2 서버 3대를 발급받아 카프카를 설치하고 console producer, console consumer로 연동한다.

apache kafka를 설치하기 위해서는 2가지 애플리케이션이 필요하다

- Zookeeper: 카프카 관련 정보를 저장하는 역할
- Kafka

### EC2 인스턴스 발급

ec2는 aws에서 발급 가능한 가상머신이다.

1. 1개의 cpu, 1기가 메모리로 구성된 t3.micro를 발급받는다.
2. 3대의 인스턴스를 발급받는다.

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/6c91c922-1988-49fe-b436-32653bfc1ddf">

### security group inbound 규칙 설정

발급 받은 ec2 보안 그룹 인바운드 규칙을 수정해야 한다.  
추가해야 할 port는 2181 / 2888 / 3888(zookeeper port), 9092(kafka 통신)

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/5abb5e56-74f5-4b0d-9b6c-d8ef0aea759b">

### /etc/hosts 설정

ec2를 발급 받았으면 .pem 파일을 생성할 수 있다. .pem 파일이 있는 경로로 이동해 아래 명령어를 입력한다.

```
chmod 400 파일명.pem
```

아래 사진처럼 인스턴스를 선택하고 연결을 클릭한다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8cc05aa0-a58c-4471-887d-25a22715ed6e">

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/7f6bab2d-11c4-4292-b076-481447529622">

맨 아래 예: ssh -i ~ 부분을 복사한다.

iterm에 새로운 프로필을 설정한다.

- 원하는 프로필 이름을 Name란에 작성한다.
- Command 콤보 박스에서 Command를 선택하고 복사한 ssh -i "파일명.pem" ~을 붙여넣기한다.  
  이때 파일명 앞에 .pem 파일이 저장되어 있는 절대 경로를 붙여줘야 한다.

만약 .pem 파일이 /Users/usuyn/test 폴더에 aws.pem 파일명으로 저장되어 있다면 아래처럼 수정한다.

```
ssh -i "/Users/usuyn/test/aws.pem" ~
```

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/57ef4f75-a282-4ad1-a548-6d6c09752052">

3개의 인스턴스 모두 위처럼 설정한다.

<img width="200" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/1b964750-30f6-4afd-b2bb-5562c10e7ce2">

인스턴스 3개에 접속해 /etc/hosts를 설정한다.

```
sudo vi /etc/hosts
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/21f1cd75-3032-4dac-810d-65e61616d2e1">

아래 내용을 추가해준다. 아래 예는 test-broker01의 /etc/hosts에 추가한 것이다.

```
// ip 주소 HostName
0.0.0.0 test-broker01
16.171.34.144 test-broker02
13.53.134.188 test-broker03
```

자신의 ip 주소는 0.0.0.0으로 설정하고 나머지는 ec2 인스턴스의 public ip 주소로 설정하면 된다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/821291bb-66e8-41c2-ab8f-593b35cab827">

### Zookeeper 설치

모든 인스턴스에 동일하게 진행한다.

```
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.8.2/zookeeper-3.8.2.tar.gz
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/a7221144-ffb0-4216-b80a-8af55fd13ff6">

다운 받은 파일의 압축을 풀어준다.

```
tar xvf zookeeper-3.8.2.tar.gz
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d8db57be-260d-46ee-a3e9-6f0a7837b990">

zookeeper의 configuration을 설정한다. zookeeper 폴더 내부의 conf 폴더에 zoo.cfg 파일을 생성한다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d2c0dd80-e839-4de9-b040-8e2ef987f9e2">

모두 동일하게 아래 내용을 작성한다.

```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=20
syncLimit=5
server.1=test-broker01:2888:3888
server.2=test-broker02:2888:3888
server.3=test-broker03:2888:3888
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e0516d25-7273-41b8-90a6-8977846539f4">

인스턴스마다 myid 파일을 생성해야 한다. 위치는 /var/lib/zookeeper/myid이며 파일 안에는 숫자 하나를 작성한다. test-broker01은 1, test-broker02는 2, test-broker03은 3이다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ed13bc7e-91ee-42f7-9eb7-177c22a62da2">

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e1b47d64-87f9-4c00-9b30-466fe7a0d265">

### Zookeeper 실행

실행 명령어를 입력했지만 ec2에 java가 설치되지 않았다고 오류가 떠서 아래처럼 openJDK 1.8을 설치한다.

```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8c05379c-ccde-49c6-8b96-6b3008d5657f">

자바를 설치한 후 아래 명령어를 통해 Zookeeper를 실행할 수 있다.

```
./bin/zkServer.sh start
```

<img width="540" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/dc455697-c8f0-4261-bb41-ac7e816d7471">

### Kafka 설치

Zookeeper를 설치할 때와 동일하게 아래 명령어를 통해 다운 받고 압축을 푼다.

```
wget https://archive.apache.org/dist/kafka/2.1.0/kafka_2.11-2.1.0.tgz

tar xvf kafka_2.11-2.1.0.tgz
```

kafka 실행을 위해 broker.id, listener, zookeeper 설정이 필요하다.  
이를 위해 kafka 폴더 내부의 config 폴더 안 server.properties를 수정한다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/c8e15131-f043-4900-819b-1d9ae7cd6278">

```
broker.id=0 // test-broker01은 0, 02는 1, 03은 2
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://test-broker01:9092 // 각 test-broker마다 01, 02, 03
zookeeper.connect=test-broker01:2181,test-broker02:2181,test-broker03/test
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/7492248c-f746-4e7e-b763-17b0cbb0386b">

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/83053f2c-22e6-4b17-882d-31174311d7cb">

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/93fea947-311e-4513-9aa3-7cbc1928f5fd">

zookeeper 설정 시 /test와 같이 route를 넣으면 zookeeper의 root node가 아닌 child node에 kafka 정보르 ㄹ저장하게 되므로 유지보수에 이득이 있다.

### Kafka 실행

아래 명령어를 통해 kafka를 실행한다.

```
./bin/kafka-server-start.sh ../config/server.properties
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/65c7df31-831e-4a72-9ac4-42854edd6438">

만약 실행 시 아래 사진처럼 error= 'Cannot allocate memory (errno=12) 오류가 발생한다면

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/a1609074-2eb2-4041-b4d1-172c2f3b08ce">

아래 명령어를 입력한 후 수정이 필요하다.

```
sudo vi kafka-server-start.sh
```

KAFKA_HEAP_OPTS의 export문을 아래처럼 수정한다.

```
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/65ab34c2-7037-4efb-8f1c-5cf9781394e4">

여기까지 진행하면 kafka 클러스터 구축 및 실행이 완료된 것이다.

### console-producer, consumer 테스트

외부망에서 정상적으로 접근이 되는지 테스트하면 kafka 클러스터가 동작하는지 확인할 수 있다.  
local machine(맥북)에서 kafka-console-producer와 kafka-console-consumer로 테스트한다.

이 과정은 이전 단계에서 kafka server를 실행시킨 상태로 진행해야 한다.

맥북에 kafka를 설치한 후

아래 명령어를 통해 새로운 topic을 생성한다.

```
./kafka-topics.sh --create --zookeeper test-broker01:2181,test-broker02:2181,test-broker03:2181/test --replication-factor 3 --partitions 1 --topic test_log
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/5b1e2741-c65e-4336-9804-87d647df0f31">

생성이 완료되면 producer와 consumer를 동시에 켜고 topic에 데이터가 정상 처리되는지 확인한다.

- producer console

```
./kafka-console-producer.sh --broker-list test-broker01:9092,test-broker02:9092,test-broker03:9092 --topic test_log
```

- consumer console

```
./kafka-console-consumer.sh --bootstrap-server test-broker01:9092,test-broker02:9092,test-broker03:9092 --topic test_log --from-beginning
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/513c9151-b19d-498c-baf7-bd17091613f0">

producer에 데이터를 입력하면 consumer에서 정상적으로 처리하는 것을 확인할 수 있다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/99078072-a355-42f8-83eb-1f324601f60b">

## Kafka Producer Application

producer는 데이터를 카프카에 보내는 역할을 수행한다.

엄청난 양의 클릭 로그들을 대량으로, 실시간으로 카프카에 적재할 때 producer를 사용한다.

### Kafka Producer의 역할

producer는 데이터를 producing 즉, 생산하는 역할을 한다. 데이터를 kafka topic에 생산한다.

- Topic에 해당하는 메시지를 생성

- 특정 Topic으로 데이터를 publish

- 처리 실패 및 재시도

### Producer Application 개발

kafka client인 consumer와 producer를 사용하기 위해서는 apache kafka 라이브러리를 추가해야 한다. 라이브러리는 gradle, maven 도구를 사용해 편리하게 가져온다.

```java
// gradle
compile group: 'org.apache.kafka', name: 'kafka-clients', version: '2.3.0'

// maven
<dependency>
		<groupId>org.apache.kafka</groupId>
		<artifactId>kafka-clients</artifactId>
		<version>2.3.0</version>
</dependency>
```

kafka client를 dependency로 잡을 때 버전을 주의해야 한다.
kafka는 broker 버전과 client 버전의 하위 호환성을 모든 버전에 대해 지원하지 않는다.

일부 kafka broker 버전은 특정 kafka client 버전을 지원하지 않을 수도 있다. broker와 client 버전의 하위 호환성에 대해 숙지하고 알맞은 kafka client 버전을 사용해야 한다.

[Kafka broker와 java client의 버젼 하위호환성 정리](https://blog.voidmainvoid.net/193)

```java
public class Producer {
  public static void main(String[] args) throws IOException {
    Properties configs = new Properties();
    configs.put("bootstrap.servers", "localhost:9092");
    configs.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

    KafkaProducer < String, String > producer = new KafkaProducer < String, String > (configs);

    ProducerRecord record = new ProducerRecord < String, String > ("click_log", "login");

    producer.send(record);
    producer.close();
  }
}
```

### Java Properties 객체를 통한 Producer 설정 및 생성

```java
Properties configs = new Properties();

configs.put("bootstrap.servers", "localhost:9092");
```

$\rightarrow$ bootstrap 서버 설정을 localhost의 kafka를 바라보도록 설정한다.

kafka broker의 주소 목록은 되도록 2개 이상의 ip, port를 설정하도록 권장된다. 둘 중 한개의 broker가 비정상일 경우 다른 정상적인 broker에 연결되어 사용 가능하기 때문이다.

그러므로 실제로 애플리케이션을 kafka와 연동할 때 2개 이상의 broker 정보를 넣는 것을 추천한다.

```java
configs.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
```

$\rightarrow$ 나머지 key와 value에 대해 StringSerializer로 직렬화 설정한다.

Byte array, String, Integer Serializer 모두 사용 가능하다.

key는 메시지를 보내면 topic의 partition이 지정될 때 쓰인다.

```java
KafkaProducer < String, String > producer = new KafkaProducer < String, String > (configs);
```

$\rightarrow$ 설정 완료한 Properties 객체로 KafkaProducer 인스턴스를 생성한다.

### ProducerRecord class 사용한 전송 객체 생성

```java
ProducerRecord record = new ProducerRecord < String, String > ("click_log", "login");
```

$\rightarrow$ key 없이 ProducerRecord 생성하는 경우

kafka client에서는 ProducerRecord class를 제공한다.

ProducerRecord 인스턴스 생성 시 어느 topic에 넣을 것인지, 어떤 key와 value를 담을 것인지 설정 가능하다.

위 코드는 key 없이 'click_log' topic에 'login'이라는 value를 보낸다.

```java
ProducerRecord record = new ProducerRecord < String, String > ("click_log", "1", "login");
```

$\rightarrow$ key를 포함해 ProducerRecord 생성하는 경우

파라미터 개수에 따라 자동으로 오버로딩되어 인스턴스 생성된다.

### 전송 및 종료

```java
producer.send(record);

producer.close();
```

전송이 이루어지며 'click_log' topic에 'login' value가 들어가게 된다.  
전송이 완료되면 close를 통해 producer를 종료한다.

## Kafka Consumer Application

데이터는 topic 내부의 partition에 저장된다. kafka consumer는 파티션에 저장된 데이터를 가져온다.
데이터를 가져오는 것을 polling이라고 한다.

### Kafka Consumer의 역할

- Topic의 partition으로부터 데이터 polling

  $\rightarrow$ topic 내부의 partition에서 메시지를 가져오는 것이다. 메시지를 가져와 특정 데이터베이스에 저장하거나 또 다른 파이프라인에 전달할 수 있다.

- Partition offset 위치 기록(commit)

  $\rightarrow$ offset(partition에 있는 데이터의 번호) 위치를 commit 한다.

- Consumer group을 통해 병렬 처리

  $\rightarrow$ consumer가 여러개일 때 병렬 처리를 할 수 있다. partition 개수에 따라 consumer를 여러개 만들면 병렬 처리가 가능해져 더 빠른 속도로 데이터를 처리한다.

### Consumer Application 개발

producer를 사용할 때처럼 라이브러리를 추가한다. 기본적으로 자바 라이브러리를 지원하며 Gradle, Maven 라이브러리 관리 도구를 사용할 수 있다.  
마찬가지로 broker 버전과 client 버전의 호환성을 확인해야 한다.

```java
// gradle
compile group: 'org.apache.kafka', name: 'kafka-clients', version: '2.3.0'

// maven
<dependency>
		<groupId>org.apache.kafka</groupId>
		<artifactId>kafka-clients</artifactId>
		<version>2.3.0</version>
</dependency>
```

```java
public class Consumer {
  public static void main(String[] args) {
    Properties configs = new Properties();
    configs.put("bootstrap.servers", "localhost:9092");
    configs.put("group.id", "click_log_group");
    configs.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    configs.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

    KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);

    consumer.subscribe(Arrays.asList("click_log"));

    while(true) {
      ConsumerRecords<String, String> records = consumer.poll(500);
      for(ConsumerRecord<String, String> record : records) {
        System.out.println(record.value());
      }
    }
  }
}
```

### Java Properties 객체를 통한 Consumer 설정 및 생성

```java
Properties configs = new Properties();
configs.put("bootstrap.servers", "localhost:9092");
```

$\rightarrow$ bootstrap servers 옵션을 통해 kafka broker를 설정한다.  
broker 중 1개에 이슈가 생기면 다른 broker가 붙을 수 있도록 여러개의 broker를 지정하는 것을 추천한다.

```java
configs.put("group.id", "click_log_group");
```

$\rightarrow$ group id를 지정한다. consumer group이라고도 불린다.

```java
configs.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
configs.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
```

$\rightarrow$ key와 value에 대한 직렬화 설정을 추가한다.

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
```

$\rightarrow$ 설정 완료한 Properties 객체로 KafkaConsumer 인스턴스를 생성한다.
이 인스턴스를 통해 데이터를 읽고 처리한다.

### 대상 topic 지정

```java
consumer.subscribe(Arrays.asList("click_log"));
```

$\rightarrow$ subscribe()를 사용해 어느 topic을 대상으로 데이터를 가져올지 설정한다. 특정 topic의 전체 partition의 데이터를 읽어온다.

```java
TopicPartition partition0 = new TopicPartition(topicName, 0);
TopicPartition partition1 = new TopicPartition(topicName, 1);

consumer.assign(Arrays.asList(partition0, partition1));
```

$\rightarrow$ assign()을 사용해 특정 topic의 일부 partition 데이터를 가져온다. key가 존재하는 데이터라면 데이터의 순서를 보장하는 데이터 처리가 가능하다.

### polling loop

```java
while(true) {
  ConsumerRecords<String, String> records = consumer.poll(500);
  for(ConsumerRecord<String, String> record : records) {
    System.out.println(record.value());
  }
}
```

$\rightarrow$ poll()이 포함된 무한루프를 사용한다.

consumer API의 핵심은 broker로부터 연속적으로 그리고 consumer가 허락하는 한 많은 데이터를 읽는 것이다. 따라서 위 loop가 consumer API의 핵심 로직이다.

consumer는 poll()을 사용해 데이터를 가져온다. 설정한 시간동안 데이터를 기다리게 된다. 위 예에서는 500ms이다.  
0.5초동안 데이터가 도착하기를 기다리고 이후 코드를 실행한다. 만약 0.5초동안 데이터가 들어오지 않으면 빈 값의 records 변수를 반환하고 데이터가 들어오면 데이터가 존재하는 records 값을 반환한다.

```java
for(ConsumerRecord<String, String> record : records)
```

records 변수는 데이터 배치로서 record의 묶음 list이다. 따라서 실제로 kafka에서 데이터를 처리할 때는 가장 작은 단위인 record로 나누어 처리한다.

records 변수를 for loop에서 반복 처리하면서 실질적으로 처리하는 데이터를 가져온다. record.value()의 반환값이 이전에 producer가 전송한 데이터이다.

### Producer에서 Consumer로 Message 전달

key가 null일 경우 2개의 partition에 round-robin으로 데이터를 넣는다.

partition으로 들어간 데이터는 partition 내에서 고유한 번호인 offset을 가지게 된다. offset은 topic, partition별로 별개 지정된다.

offset은 consumer가 데이터를 어느 지점까지 읽었는지 확인하는 용도로 사용된다.

consumer가 데이터를 읽기 시작하면 offset을 commit하게 되는데 이렇게 가져간 내용의 정보는 kafka의 \_\_consumer_offset topic에 저장한다.

<img width="550" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e0530eb7-abf5-4d94-83f2-6456217a3619">

partition #0의 데이터를 3번 offset까지 읽고 partition #1의 데이터를 2번 offset까지 읽은 consumer가 실행 중지되었다가 재실행된다면 \_\_consumer_offset을 통해 중지되었던 시점을 알고 있으므로 다시 처리 시점을 복구할 수 있다. (고가용성의 특징을 가진다.)

### Multiple Consumer

partition이 2개라고 가정할 때

- **Consumer가 1개인 경우**

  <img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/0cfa90f9-0496-4f5e-9fcb-a959ce93f434">

  $\rightarrow$ 2개의 partition에서 데이터를 가져간다.

- **Consumer가 2개인 경우**

  <img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8fff8c13-9c81-4e57-a172-e491c4bd1039">

  $\rightarrow$ 각 consumer가 각각의 partition을 할당해 데이터를 처리한다.

- **Consumer가 3개 이상인 경우**

  <img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/0ca09e27-79da-48ce-a5f6-f965eb906487">

  $\rightarrow$ 이미 partition들이 각 consumer에 할당되어 더이상 할당될 partition이 없어 동작하지 않는다.

  여러 partition을 가진 topic에 대해 consumer를 병렬 처리하고 싶다면 consumer는 partition 개수보다 적거나 같아야 한다.

### Different Groups

각기 다른 그룹에 속한 consumer들은 다른 consumer 그룹에 영향을 미치치 않는다.

데이터 실시간 시각화 및 분석을 위한 consumer 그룹과 데이터 백업을 위한 consumer 그룹이 있을 때 각 그룹이 동일한 topic의 데이터를 읽어도 서로 영향을 미치지 않는다.

\_\_consumer_offset topic에는 consumer 그룹별로, topic별로 offset을 나누어 저장하기 때문이다.

따라서 하나의 topic으로 들어온 데이터는 다양한 역할을 하는 consumer들에 의해 각자 원하는 대로 처리될 수 있다.

## Kafka Streams Application

consumer를 사용해서 데이터를 처리하는 것보다 더 안전하고 빠르면서도 다양한 기능을 사용할 수 있는 Kafka Streams이다.

kafka에서 공식적으로 제공하는 자바 라이브러리이다. topic에 있는 데이터를 낮은 지연과 함께 빠른 속도로 처리할 수 있다.

streams는 라이브러리로 제공되기 때문에 Java, Scala, Kotlin 등 JVM 기반 언어로 개발할 수 있다.

### Kafka Streams의 장점

- **카프카와 완벽 호환된다.**

  $\rightarrow$ streams는 매번 kafka가 릴리즈될 때마다 kafka cluster와 완벽하게 호환되어 최신의 기능을 가지고 있다.

  kafka에 보안 기능이나 ACL 같은 것들이 붙어있어도 완벽하게 호환된다.

  또한 유실이나 중복 처리되지 않고 딱 한번만 처리할 수 있는 강력한 기능을 가지고 있다. kafka와 연동하는 이벤트 프로세싱 도구 중 거의 유일하다고 볼 수 있다.

- **스케쥴링 도구가 필요없다.**

  $\rightarrow$ kafka와 연동하는 스트림 프로세싱 툴로 가장 많이, 널리 사용하는 것이 Spark Streaming이다. Spark Streaming 또는 spark structured streaming을 사용하면 kafka와 연동하여 마이크로 배치 처리를 하는 이벤트 데이터 애플리케이션을 만들 수 있다.

  문제는 스파크를 운영하기 위해서 yarn이나 mesos와 같이 cluster 관리자 또는 리소스 매니저 같은 것들이 필요하며 cluster 운영을 위해 대규모 장비들을 구축해야 한다.  
  반면에 streams를 사용하면 스케쥴링 도구는 전혀 필요없다.

  streams 애플리케이션은 consumer 애플리케이션이나 WAS 애플리케이션을 배포하는 것처럼 원하는 만큼 배포하면 된다.

- **Streams DSL과 Processor API를 제공**

  $\rightarrow$ streams를 구현하는 방법은 2가지이다. 대부분의 경우에는 Streams DSL을 사용해서 해결할 수 있다.  
  Streams DSL은 이벤트 기반 데이터 처리를 할 때 필요한 기능인 map, join, window 등과 같은 메서드들을 제공하기 때문에 사용하기 편하다.

  Streams DSL에 없는 기능이 있다면 Processor API를 사용해 로직을 작성하면 된다.

  Streams DSL만이 제공하는 KStream, KTable, GlobalKTable은 독특한 스트림 처리 개념이다.  
  스트림 데이터 처리뿐만 아니라 대규모 key-value 저장소로도 사용할 수 있는 기능 또한 가지고 있다.

- **로컬 상태 저장소를 사용한다.**

  $\rightarrow$ 실시간으로 들어오는 데이터를 처리하는 방식은 크게 비상태 기반 처리, 상태 기반 처리 2가지이다.

  Stateless라고도 불리는 비상태 기반 처리는 필터링이나 데이터를 변환하는 처리이다.  
  데이터가 들어오는 족족 바로 처리하고 프로듀스하면 되기 때문에 유실이나 중복이 발생할 염려가 적고 쉽게 개발할 수 있다.

  Stateful이라고도 불리는 상태 기반 처리 직접 구현은 어렵다. window, join, aggregation과 같은 처리는 이전에 받았던 데이터를 프로세스가 메모리에 저장하고 있으면서 다음 데이터를 참조해서 처리해야 하기 때문이다. 이러한 어려움을 극복하게 도와주는 것이 바로 streams이다.

  streams는 이런 어려운 처리를 돕기 위해 로컬에 RocksDB를 사용해 상태를 저장하고 이 상태에 대한 변환 정보는 kafka의 변경 로그(changelog) topic에 저장한다.  
  streams를 사용하면 프로세스에 장애가 발생하더라도 그 상태가 모두 안전하게 저장되기 때문에 자연스럽게 장애 복구가 가능하다.

### Streams DSL이 제공하는 이벤트 기반 메서드

```java
KStream<String, String> paymentStream = builder.stream("payment");
KStream<String, String> filteredStream = paymentStream
                            .filter((key, value) -> key.equals("unknown"));
filteredStream.to("unknown-payment");
```

위 코드는 payment topic에 message key가 "unknown"인 데이터를 필터링해서 unknown-payment topic으로 보내는 streams 코드이다.

기존처럼 consumer로 polling하거나 producer를 어렵게 구현할 필요가 없다. 위처럼 Streams DSL이 제공하는 이벤트 기반 메서드를 사용하면 쉽게 구현할 수 있다.
