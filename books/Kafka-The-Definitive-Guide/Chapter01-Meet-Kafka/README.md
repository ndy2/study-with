---
tags:
  - kafka
title: CHAPTER 1 Meet Kafka
author: ndy2
date: 2024-01-22
description: ""
---
> [!quote]
> *Every enterprise is powered by data.*

# Publish/Subscribe Messaging

***Pub/Sub messaging*** 이란 sender (publisher) 가 데이터 (message) 를 recevier 에게 직접 전달하지 않는 특성을 가지는 패턴이다.

Figure 1-1. A Single, direct metrics publisher

Figure 1-4. Multiple publish/subscribe systems

`1-4` 의 구조는 분명 `1-1` 의 구조 보다 고도화 되었고 couping 을 줄였지만 장점만 있는 것은 아니다.
중복이 많아지고 다양한 component 가 등장함에 따라 각각 장애가 발생할 가능성을 가지게 되었다.

# Enter Kafka

Apache Kafka 는 이런 문제를 해결하기 위해 디자인 된 publish/subscribe messaging 시스템이다.

## Messages and Batches

- `message` 
	- The **unit of data** within Kafka
	- 데이터베이스의 관점에서는 `row` 혹은 `record` 와 유사한 개념이다.
	- 메시지란 간단히 Kafka 가 관심있어 하는 바이트 배열 (an array of bytes)이다.
	- `key` 는 `message` 가 가지는 metadata 로서 또 다른 바이트 배열이다.
	- `key` 는 `meesage` 를 partition 에 작성하는 방법을 더 정교하게 제어하기 위해 활용한다.
		- e.g.) key 를 hash 떠서 modulo 연산을 통해 publish 대상 partition 을 정한다.
	- `key` 에 대한 더 자세한 내용은 Chapter 3 에서 다룬다.
	- Kafka 는 `message` 와 `key` 의 내용에는 아무런 관심이 없다. 

- `batches`
	- `batch` 란 동일한 `topic` 과 `partition` 에 발행되는 collection of `messages` 이다.
	- `batch` 단위로 `message` 를 발행함으로써 network roundtrip 을 줄여 효율적으로 데이터를 작성할 수 있다.
	- 일반적으로 `message` 는 `batch` 단위로 압축되어 단순히 roundtrip 을 줄이는 것 이상의 이점을 제공한다.
	- 하지만 이는 당연히 Latency 와 Throughput 사이의 Trade-Off 를 낳는다.


## Schemas

> [!quote] 참고 자료
> * [Confluent Documentation > Schema Management > Overview > Understanding Schemas](https://docs.confluent.io/platform/current/schema-registry/index.html)

위에서 이야기 했든 Kafka 는 message 의 내용에 아무 관심이 없고 그저 byte array 로 취급한다. 하지만 `schema` 라는 추가적인 구조를 통해 message 의 내용이 사용자에게 더 쉽게 이해되도록 하는 것은 좋은 방법이다.

- Json, XML 는 Human-Readable 한 Schema 이다.
- 하지만 이들은 type handling, version system 등의 기능을 잘 지원하지 않는다.
- 많은 Kafka 개발자 들은 Apache Avro 라는 직렬화 프레임워크를 통해 메시지의 형태를 정의한다.

## Topics and Partitions

- Kafka 의 `message` 는 `topic` 으로 분류된다.
- `topic` 은 데이터베이스의 테이블 혹은 파일시스템의 폴더와 유사한 개념이다.

- `topic`은 나아가 `partition`으로 나뉜다.
- Kafka 는 `partition` 을 통해 redundancy 와 scalability 를 제공한다.
- `partition` 은 서로 다른 server 에서 hosted 될 수 있다.

> [!note]
> * `topic` 은 일반적으로 여러 `partition` 으로 구성된다.
> * 일반적으로 토픽은 여러 파티션으로 나뉘어져 있기 때문에 전체 토픽을 통한 메시지 시간 순서가 보장되지 않는다. 단일 파티션 내에서만 시간 순서가 보장된다.


## Producers and Consumers 

Kafka Clients
- **producers**
	- a.k.a. `publishers` or `writers`
	- create new `message`
- **consumers**
	- a.k.a `subscribers` or `readers`
	- read `messages`
	- consumer 는 message 의 offset 을 추적하여 어떤 메시지가 이미 consume 되었는지 관리한다.
	- producer 는 메시지를 생성할때 항상 파티션 별로 unique 하게 증가하는 offset 을 message 에 추가한다.
	- 각 파티션에는 마지막으로 consume 된 message 의 offset 을 Zookeeper 혹은 Kafka 를 통해 기록함으로써 consumer 는 위치를 잃지 않고 중단 혹은 재 실행 할 수 있다.
	- consumer 는 또한 `consumer group` 으로 함께 동작한다.
- Kafka Connect API
	- Data integration
- Kafka Streams
	- stream processing



