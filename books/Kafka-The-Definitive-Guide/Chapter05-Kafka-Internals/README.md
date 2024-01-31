---
tags:
  - kafka
title: CHAPTER 5 Kafka Internals
author: ndy2
date: 2024-01-28
description:
---
> [!quote] 참고 자료
> * confluent 에서 kafka internals 에 대한 강의를 공개하였다. - https://cnfl.io/48CZZZT

Kafka 의 내부 구조를 이해하는 것은 Kafka 를 production 에서 사용하거나 Kafka 를 이용해 애플리케이션을 개발하는것에 꼭 필요하지는 않다. 이 책에서는 Kafka 실무자와 연관이 큰 세가지 주제에 대해 Kafka 의 내부구조를 살펴본다.

- How Kafka replication works
- How Kafka handles request from producers and consumers
- How Kafka handles storage such as file format and indexes

# Cluster Membership

Kafka 는 현재 클러스터의 브로커 목록을 관리하기 위해 **Apach Zookeeper** 를 활용합니다. Kafka 브로커는 매 프로세스 실행시 자신의 ID 를 Zookeeper 에 *ephemeral node* 로 등록합니다. 또한 Kafka 브로커는 Zookeeper 의 `/brokers/ids` 경로를 구독하여 브로커의 추가/제거를 알 수 있습니다. 

Zookeeper Node 에 브로커의 ID 가 사라지더라도 `각 토픽의 레플리카 목록`과 같은 다른 자료구조에 브로커의 ID 가 남아 있을 수 있습니다. 이 경우 기존의 broker id 와 같은 새로운 broker 를 시작한다면 즉시 cluster 에 브로커를 추가 할 수 있습니다.

# The Controller

Controller 는 브로커중 한 녀석입니다. 기본적인 브로커의 역할과 함께 `partition leaders` 를 선정하는 역할을 가집니다. 클러스터 중 첫번째로 시작하는 broker 가 controller 로 결정이 되며 Zookeeper 의 `/controller` 경로에 ephemeral node 를 추가합니다.

다른 broker 도 시작할 때 `/controller` 경로에 노드를 추가하려고 하지만 `"node already exists"` 예외를 받습니다. 이때 다른 broker 들은 이미 클러스터 내에 controller 가 생성되었다고 판단하고 controller node 를 `zookeeper watch` 하여 해당 노드의 변경을 확인합니다.

controller 브로커가 중단되거나 Zookeeper 와 연결이 끊기면 ephemeral node 가 제거 됩니다. 이때 다른 브로커는 해당 노드를 `watch` 하고 있었기 때문에 controller 가 제거 되었음을 알고 자신을 새로운 controller 로 등록하고자 시도합니다. 처음 등록 단계와 마찬가지로 그 중 한 노드만이 새로운 ephemeral node 를 등록 하고 새로운 controller 로 선정 됩니다. 

이런식으로 새로운 controller 가 생성 될 때 마다 controller 는 더 높은 `controller epoch` 를 가지게 됩니다.

Controller 가 브로커가 클러스터에서 제거되었음을 확인하면 해당 브로커가 담당하던 partition 에 대한 새로운 leader 를 선정합니다.

# Replication

Replication 은 Kafka architecture의 핵심입니다. Kafka documentation 의 첫 문장에서 Kafka 는 자신을 "a distributed, partitioned, replicated commit log service." 라고 설명합니다. Replication 을 통해 Kafka 는 가용성, 내고장성을 보장합니다.

이전에 이야기 했듯이 Kafka 의 데이터는 topic 으로 구조화됩니다. 각 Topic 은 partition 으로 나뉩니다. 각 Partition 은 다수의 replica (복제본) 를 가집니다. replica 는 여러 broker 에 저장됩니다. 일반적으로 각 broker 는 여러 topic, partition 의 수백/ 수천개의 replica 를 저장합니다.

Replica 에는 두가지 종류가 있습니다.

*`Leader Replica`*
- 각 partition 은 하나의 replica 를 leader 로 선정합니다.
- 모든 producer 와 consumer 의 요청은 leader 에게 전달 됩니다. 이를 통해 consistency 를 보장 할 수 있습니다.

*`Follower replica`*
- 리더각 아닌 replica 를 follower 라 부릅니다. Follower 는 요청을 직접 전달 받지 않습니다. Follower 는 오직 Leader 의 메시지를 복제하며 Leader 와의 동기화를 유지하고자 합니다.
- Leader replica 에 장애가 생기면 Follower 중 한 replica 는 Leader 로 승진합니다.

> [!note] Terminologies
> * Broker Controller 브로커와 그렇지 않은 (일반) 브로커로 구분합니다.
> * 반면 Partition 의 복제본 (replica) 는 Leader 와 Follower 로 구분합니다.

Leader replica 는 또한 어떤 Follower replica 가 자신과 동기화 된 상태인지 확인합니다. Follower Replica 는 Leader 와 동기화된 상태를 유지하기 위해 leader 에게 **`Fetch`** 요청을 보냅니다. 이 Fetch 요청에는 Replica 가 해당 partition 에 대해 받고자 하는 offset 값이 순차적으로 포함됩니다. 이를 통해 Leader replica 는 각 follower 의 현재 동기화 여부 및 정도를 파악 할 수 있습니다.

Follower Replica 가 10초 이상 Leader 를 f/u 하지 못했다면 해당 replica 는 *`out of sync`* 하다고 합니다. 이런 뒤쳐진 replica 들은 leader 에 장애가 발생하여 다음 leader 를 선정해야 하는 상황에서 후보가 될 수 없습니다. 반대로 leader 를 잘 f/u 하는 replica 는 `in-sync replicas` 라고 합니다.

위에서 이야기한 10초라는 값은 정확히 `replica.lag.time.max.ms` 값으로 결정 됩니다.

추가적으로 *`preferred leader`* 라는 개념도 있습니다.
preferred leader 란 토픽이 최초 생성된 시점의 leader 입니다. preferred leader 가 available 하다면 가능하면 이를 이용하는 것이 좋습니다. `auto.leader.rebalance.enable` 속성을 통해 preferred leader 가 현재 leader 가 아니라면 rebalance 를 수행 할 수 있습니다. (default - true)

# Request Processing

Kafka 의 모든 요청은 다음과 같은 표준 헤더를 가집니다.
- Request Type (a.k.a. API key)
	- produce, fetch, metadata ...
- Request version
- Correlation ID
- Client ID

Replication 절에서 설명 했든 Kafka 의 모든 요청은 Leader 에서 처리 되어야 합니다. 요청이 도착한 broker 가 해당 partition 에 대한 leader 를 가지고 있지 않다면 이 요청은 leader 를 가진 broker 에게 위임 되어야 합니다.

이때 카프카 브로커가 리더를 가진 브로커를 판단하기 위해 자신 이외의 모든 브로커에게 *metadata request* 를 보내 해당 토픽의 parition 에 대한 leader 를 포함하는지 여부를 확인 합니다. 이 정보는 각 topic metadata 로써 브로커에 캐싱 됩니다.

# Physical Storage

Kafka 에서 물리적 데이터 저장의 기본 단위는 partition replica 입니다. 하나의 partition 은 여러 broker 에 나누어 저장 될 수 없으며 심지어 같은 broker 내에서도 같은 disk (drive) 에 저장 되어야 합니다.

## Partition Allocation

Topic 을 생성하면, Kafka 는 먼저  replica 를 물리적으로 어떻게 분배할 지 결정합니다. 요약하자면 최대한 섞어서 같은 partition 을 분배합니다.