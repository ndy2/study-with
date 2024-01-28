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

> [!todo] 
> To Be Done

# Request Processing

> [!todo] 
> To Be Done

# Physical Storage

> [!todo] 
> To Be Done