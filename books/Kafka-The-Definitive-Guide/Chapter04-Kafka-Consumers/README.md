---
tags:
  - kafka
title: CHAPTER 4 Kafka Consumers
author: ndy2
date: 2024-01-27
description: ""
---
# Consumer Overview

Kafka 가 데이터를 읽는 법을 이해하기 위해서는 먼저 consumer 과 consumer group 을 이해해야합니다.

## Consumer and Consumer Groups

Producer 가 생산 속도가 너무 빨라 Consumer 가 처리 할 수 없다면 어떨까요? 이런 경우를 대비해 마치 여러개의 Producer 가 하나의 topic 에 데이터를 쓸 수 있는 것 처럼 Consumer 는 여러개의 Consumer 가 group 을 구성하여 하나의 topic 을 처리 할 수 있습니다. 이를 `consumer group` 이라고 부릅니다. 

그림을 통해 살펴보겠습니다.


![[consumer-group-1.excalidraw]]

네개의 파티션으로 구성된 Topic T1 의 모든 Partion 을 처리하는 Consumer Group 1 의 Consumer C1 의 그림입니다. 다른 Consumer 3개 추가하여 각 Consumer 가 하나의 파티션을 다루도록 아래 그림 처럼 구성할 수 도 있습니다.

![[consumer-group-3.excalidraw]]

만약 하나의 Consumer 를 추가하여 파티션 숫자보다 많은 Consumer 를 구성한다면? 그 Consumer 는 메시지를 읽지 못하고 그냥 놀기만 할 것입니다.

![[consumer-group-4.excalidraw]]
토픽의 파티션 숫자보다 더 많은 Consumer 를 한 Consumer Group 에서 Topic 에 구독을 해도 아무 소용이 없다는 사실을 명심하세요.

하지만 다른 Consumer Group 을 추가 하는 것은 가능 합니다. 물론 Consumer Group 1 과 Consumer Group 2 가 처리 하는 데이터는 중복 됩니다.


## Consumer Groups and Partition Rebalance

### Partition Rebalance

위에서 살펴보앗듯이, 같은 consumer group 에 포함된 consumer 는 구독하는 topic 의 파티션에 대한 소유권을 공유합니다. (모두 합쳐 파티션을 가진 다는 뜻)

group 에 새로운 consumer 를 추가하면 이전에 다른 consumer 가 처리하던 partition 을 처리해야 합니다. consumer 에 장애가 발생해 숫자를 줄여야 하는 경우도 마찬가지 입니다. 장애가 난 consumer 가 처리하던 partition 을 정상 consumer 들이 나누어 처리 해야 합니다. 

이렇게 파티션의 소유권이 한 consumer 에서 다른 consumer 로 이동하는 과정을 `rebalance` 라고 합니다. rebalance 를 통해 consumer group 에 고가용성과 확장성을 제공할 수 있습니다. 하지만 rebalance 중 consumer 는 메시지를 처리 할 수 없습니다.

### HeartBeats

consumer 는 `heartbeat` 를 `group coordinator` 에 해당하는 Kafka broker 에게 보내는 것으로 consumer group 내에서 자신이 살아있음을 알리고 담당 partition 에 대한 소유권을 주장합니다.

heartbeat 는 consumer poll 그리고 record commit 시에 발생합니다.

consumer 가 충분히 오랜시간 동안 heartbeat 를 보내지 않으면 group coordinator 는 해당 consumer 가 죽었다고 판단하고 rebalance 를 시작합니다.
# Kafka Consumer 생성

KafkaConsumer 를 생성하는 과정은 KafkaProducer 를 생성하는 것과 유사합니다. 마찬가지로 필수 속성값 `bootstrap.servers`, `key.deserializer` 그리고 `value.deserilizer` 를 가집니다. 


# Topic 구독

KafkaConsumer 를 생성하였다면 토픽을 구독할 수 있습니다.

놀라운점은 다수의 토픽을 regex 기반으로 구독할 수 있다는 점입니다. regex 에 매칭되는 토픽이 추가되면 subscriber 는 거의 즉시 해당 topic 을 소모하기 시작합니다. regex 기반의 토픽 구독은 대부분 Kafka 와 다른 시스템간 데이터를 replicate 하는 용도로 활용됩니다.

# The Poll Loop

consumer API 의 가장 중요한 부분은 server 로 부터 데이터를 poll 하는 loop 입니다.
```scala
object PingConsumer extends App {  
  private val consumer = new KafkaConsumer[String, String](CONSUMER_CONFIG)  
  try {  
    consumer.subscribe(Seq(TOPIC).asJava)  
    while (true) {  
      println(s"consumer.poll")  
      val records = consumer.poll(java.time.Duration.ofMillis(10000))  
      records.iterator().asScala.foreach { record =>  
        println(s"Received message: record=$record")  
      }  
    }  
  } catch {  
    case e: Exception => println("Exception occurred in PingConsumer: " + e)  
  } finally {  
    println("consumer.close()")  
    consumer.close()  
  }  
}
```

# Consumer 설정


> [!quote] 참고 자료
> * [kafka > documentation > Consumer Configs](https://kafka.apache.org/documentation.html#consumerconfigs)

`fetch.min.bytes`

records 를 조회 할 때 브로커로부터 얻고자하는 최소 데이터 크기를 명시합니다. 브로커가 데이터를 받았지만 이 값보다 적다면 Consumer 는 이 값 이상의 데이터가 쌓일 때 까지 대기합니다.

`fetch.max.wait.ms`

위 속성으로 수행하는 대기에 대하여 최대 대기 시간을 정의합니다. 기본적으로 Kafka 는 500 ms 의 대기 시간을 가집니다. 즉, 충분한 데이터가 있지 않을 때 Kafka 는 기본적으로 500 ms 대기를 수행합니다.

`session.timeout.ms`

consumer 가 heartbeat 를 보내지 않고 살아 있을 수 있는 시간을 정의합니다. 기본은 3초. 이 값은 `heartbeat.interval.ms` 과 아주 연관 되어 있는데 `heartbeat.interval.ms` 값은 `poll()` 메서드가 발생시키는 heartbeat 의 frequency 를 조절합니다. 둘은 보통 함께 바뀌며 `heartbeat.interval.ms` 는 `session.timeout.ms` 의 1/3 정도를 가집니다.


`auto.offset.reset`
	`"latest"` -> consumer 가 실행된 시점 이후의 record 를 읽음
	`"earliest"` -> 파티션의 모든 데이터를 처음 부터 읽음

`enable.auto.commit`
	`true` -> offset 을 자동으로 commit 한다.
	`false` -> 자동 커밋을 하지 않는다.

`max.poll.records`
한번의 `poll()` 호출을 통해 조회할 record 개수의 최댓값을 설정한다.

# Commits and Offsets

# Rebalance Listeners

# 특정한 offset 의 Record 를 읽기

# Exit Poll Loop Cleanly

# Deserializers

# Consumer without Consumer Group

