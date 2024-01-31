---
tags:
  - kafka
title: CHAPTER 6 Reliable Data Delivery
author: ndy2
date: 2024-01-30
description: ""
---
신뢰성은 시스템의 중요한 속성이며 한사람이 아닌 시스템의 모두가 함께 고민해야하는 문제이다. Kafka 관리자, Linux 관리자, 네트워크, storage 관리자와 애플리케이션 개발자 모두가 함께 안정성있는 시스템을 구축하기위해 노력해야한다.

신뢰성은 대체로 속도/간결성과 trade-off 관계를 가지며 kafka 와 client API 는 이를 충분히 제어할 수 있도록 설정과 유연성을 제공한다. 이런 유연성 때문에 안정적이지 않은 상태의 시스템임에도 Kafka 만 믿고 안정적이라고 잘 못 판단 할 수 있다.

# Reliability Guarantees

신뢰성 보장을 전통적인 RDBMS 관점에서 생각하보자. RDMBS 는 ***ACID*** 를 보장한다. 그렇다면 Kafka 는 어떨까? 오늘날 Kafka 와 같은 "Distributed, partitioned, replicatded commit log service" 는 자신이 제공하는 신뢰성을 어떻게 정의하며 많은 사용자를 끌어 모았을까?

- Kafka 는 파티션 내 메시지에 대한 순서를 보장합니다.
- 발행된 메시지는 모든 in-sync replica 의 파티션에 작성되었을때 `commit` 되었다고 여겨집니다.
- `commit` 된 메시지는 단 하나의 replica 만 살아있다면 사라지지 않습니다.
- Consumer 는 `commit` 된 메시지 만을 읽을 수 있습니다.


> [!question]
> * commit 은 이미 fetch 한 데이터를 어디까지 읽었다고 메시지를 남기는 것 아닌가?
> * 여기서 말하는 commit 은 의미가 좀 다른 것 같다.


# Replication

Replication 메커니즘은 Kakfa Reliability 에서 가장 중요한 개념입니다. Replication 에 대해선 [[books/Kafka-The-Definitive-Guide/Chapter05-Kafka-Internals/README|Chatper05-Kafka-Internals]] 에서 자세하게 다루었습니다. 잠깐 중요한 내용을 복습합시다.

> [!note] Let's recap the highlights of Kafka Replication
> * Kafka 의 topic 은 `partition` 으로 나뉜다. partition 은 카프카 데이터의 basic building block 이며 한 broker 의 한 disk 에 저장된다.
> * Kafka 는 한 파티션 내의 이벤트 (메시지)에 대한 순서를 보장한다.
> * 한 topic 을 구성하는 partition 은 하나의 leader partition 과 그렇지 않은 follower partition 으로 나뉜다.
> * 데이터의 읽기/쓰기 작업은 모두 leader 를 통해 이루어지며 follower 은 leader 에 대한 sync 를 유지한다.



# Reliable Broker

## `replication.factor`

> [!todo] 
> To Be Done

## `unclean.leader.election.enable`

> [!todo] 
> To Be Done

## `min.insync.replicas`

> [!todo] 
> To Be Done

# Reliable Producer

## `acks`

`acks=0`
프로듀서는 메시지를 전송하면 바로 성공으로 간주합니다.
네트워크를 타기만 하면 성공으로 간주합니다.

`acks=1`
프로듀서가 메시지를 보낼 때 최소한 한 개의 replica (leader replica)에 메시지를 성공적으로 복사한 후에 성공으로 간주합니다.

`acks=all`
모든 in-sync replica 에 메시지를 성공적으로 전달 한 다음 프로듀서의 전송을 성공으로 간주합니다.

## Retry 를 설정하기

> [!todo] 
> To Be Done

# Reliable Consumer

> [!todo] 
> To Be Done

# Validating System Reliability

자신에게 필요한 Reiablity 수준을 확인하고 이를 Broker/ Producer 그리고 Consumer 의 설정에 반영했다면 전체 시스템의 안정성을 다음과 같이 검증 해 봅시다.

## 설정 검증

Broker 와 Client 의 설정을 애플리케이션 로직과 분리하여 테스트 하면 쉽습니다. Kafka 는 `org.apache.kafka.tools` 패키지의 `VerifiableProduction`/ `VerifiableConsumer`클래스를 통해 이런 기능을 제공합니다. 

설정 검증을 통해 테스트 할 수 있는 시나리오에는 다음과 같은 예시가 있습니다.

1. Leader election
2. Controller election
3. Rolling restart
4. Unclean leader election


## 애플리케이션 검증

Broker 와 Client 의 설정이 요구사항을 만족한다고 판단되면 이제 application 이 이를 보장하는지 확인 할 차례입니다. 이는 custom 예외 처리 구문/ offset 에 대한 commit 등을 포함합니다.

## Production 에서의 안정성 모니터링

> [!todo] 
> To Be Done