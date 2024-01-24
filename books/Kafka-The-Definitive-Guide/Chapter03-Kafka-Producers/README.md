---
tags:
  - kafka
title: CHAPTER 3 Kafka Producers
author: ndy2
date: 2024-01-23
description: ""
---
# Producer Overview

## 1. `ProducerRecode` 생성

- 메시지를 발행 하는 것은 `ProducerRecord` 를 생성하는 것으로 부터 시작됩니다.
- `ProducerRecord` 는 대상이 되는 `토픽 - topic` 과 전달하고자 하는 `값 - value` 을 포함합니다.
- `ProducerRecord` 를 `전송 -send` 하면 producer 는 바로 key 와 value objects 를 ByteArray 로 `직렬화 - Serialize` 하여 네트워크에 전송할 수 있도록 합니다.

## `partitioner` 에 데이터 전송

- `ProducerRecord` 에 파티션을 지정하였다면 