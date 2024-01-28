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

## 2. `partitioner` 에 데이터 전송

- `ProducerRecord` 에 파티션을 지정하였다면 Partitioner 는 아무것도 하지 않고 지정한 파티션을 반환합니다.
- 그렇지 않다면, partitionaer 는 파티션을 선택합니다. 보통은 ProducerRecord 의 key 값을 이용해 파티션을 선택합니다.
- 파티션이 선택되었다면 프로듀서는 Record 가 전달될 topic 과 partition 을 모두 알았다는 것입니다. 따라서 동일한 topic, partition 으로 발행된 record 들을 모아놓은 batch 에 대상 Record 를 추가합니다.
- 이후 별도의 스레드를 이후에 배치 단위로 모인 record 를 적절한 Kafka 브로커에게 전송합니다.

## 3. `RecordMetadata` 수신

- 브로커가 메시지를 전달 받으면 응답을 보냅니다.
- Kafka Broker 가 성공적으로 메시지를 썻다면 `RecordMetadata` 를 반환합니다.
	- RecordMetadata 에는 topic, partition, offset 에 대한 정보가 포함됩니다.
- Kafka Broker 가 메시지를 쓰는것에 실패 했다면 error 를 반환합니다.
	- producuer 가 예외를 받으면 몇번 재시도 하고 그 이후에는 error 를 반환합니다.

# Kafka Producer 생성

Kafka 에 메시지를 발행하는 가장 첫 단계는 producer 를 적절한 설정으로 생성하는 것입니다. Kafka Producuer 는 세가지 필수 속성을 가집니다.

`bootstrap.servers`

- Kafka cluster 와 최초 연결을 맺기위한 `host:port` 의 모음입니다.
- 반드시 브로커의 모든 주소를 가질 필요는 없습니다.
- 하지만 가용성을 위해 두개 이상의 host:port 를 가지는 것을 추천합니다.

`key.serializer`
- records 의 key 를 직렬화 하기 위해 사용하는 클래스의 이름입니다.
- `key.serializer` 값은 항상 `org.apache.kafka.common.serialization.Serializer` 를 구현한 클래스의 이름을 가져야합니다.
- Kafka client 는 기본 구현체인 ByteArraySerilizer, StringSerializer 와 IntegerSerizlier 등을 제공합니다.
- `key,serilizer` 값은 Producer 가 Key 를 사용하지 않을 때 도 필요합니다.


`value.serilizer`
-  records 의 value 를 직렬화 하기 위해 사용하는 클래스의 이름입니다.
-  `value.serializer` 값은 항상 `org.apache.kafka.common.serialization.Serializer` 를 구현한 클래스의 이름을 가져야합니다.

# 메시지 전송하기

> [!note] 메시지 전송의 세가지 방식
> 1. *Fire-and-forget* - 쏘고 잊어라!
> 메시지를 전송하고 성공적으로 도착했는지 여부는 신경쓰지 않습니다.
> 
> 2. *Synchronous send* - 동기적 전송
> `send()` 가 반환한 `Future` 객체에 바로 `get()` 을 호출하여 동기적으로 성공 여부를 확인 합니다. 
> 
> 3. Asynchronous sned - 비동기적 전송
> `send()` 메서드를 callback 함수와 함께 호출합니다. callback 은 Kafka 브로커로의 응답을 받으면 실행 됩니다.


# Producuer 설정


> [!quote] 참고 자료
> * [kafka > documentation > Producuer Configs](https://kafka.apache.org/documentation.html#producerconfigs)


`acks`
`acks` 는 레코드가 데이터를 성공적으로 작성 했다고 판단하기 위해 몇개의 partition replicas 에 응답을 받아야 하는가를 조절합니다. 이 설정은 메시지의 loss 에 큰 영향을 줍니다. `acks` 는 `0`, `1` 혹은 `all` 을 가집니다.

- acks = 0
	producer 는 성공적인 메시지 전송의 판단을 위해 응답을 대기하지 않습니다.
	메시지가 잘못되어도 신경 쓰지 않지만 대기 시간을 줄임으로써 높은 처리량을 달성할 수 있습니다.

- acks = 1
	producer 는 leader 레플리카로 부터 응답을 받으면 성공적인 전송이라고 판단합니다.

- acks = all
	모든 in-sync 레플리카로 부터 응답을 받으면 성공적인 메시지라고 판단합니다. 

`buffer.memory`
producer 가 `send()` 를 처리할 수 있는 속도보다 더 빠르게 호출 될 시 사용할 버퍼의 크기를 설정합니다.

`compression.type`
메시지의 압축 알고리즘을 설정합니다. 기본적으로 메시지는 압축되지 않습니다. `snappy`, `gzip` 혹은 `lz4` 로 설정 할 수 있습니다. 메시지 압축을 통해 CPU 의 사용은 늘어나지만 더 적은 네트워크 bandwidth 를 이용할 수 있습니다.

`retries`
producer 가 서버로부터 retry 하면 해결될 수 도 있는 에러를 받은 경우 재시도 할 횟수를 정의합니다.

`batch.size`
여러 메시지가 동일한 파티션으로 전송될 때 batch 될 크기를 설정합니다. 메시지의 숫자가 아닌 메모리 크기입니다.


# Serializers

## Custom Serializers

일반적으로 Custom Object 에 대한 Serizlier 를 제공할 때도 Avro, Thrift 혹은 Protobuf 와 같은 generic sereializer 를 이용하는 것이 좋습니다. 그 이유를 알아보기 위해 먼저 Custom Serializer 를 이용해봅시다.

```java
public class Customer {
	 private int customerID;
	 private String customerName;
	 public Customer(int ID, String name) {
		 this.customerID = ID;
		 this.customerName = name;
	 }
	 public int getID() {
		 return customerID;
	 }
	 public String getName() {
		 return customerName;
	 }
}
```


```scala
class CustomerSerializer extends Serializer[Customer] {  
  
  override def serialize(topic: String, data: Customer): Array[Byte] = {  
    try {  
      val serializedName = data.name.getBytes("UTF-8")  
      val stringSize = serializedName.length  
  
      val buffer = ByteBuffer.allocate(4 + 4 + stringSize)  
      buffer.putInt(data.id)  
      buffer.putInt(stringSize)  
      buffer.put(serializedName)  
  
      buffer.array()  
    } catch {  
      case e: Exception => throw new SerializationException("Error when serializing Customer to byte[] " + e);  
    }  
  }  
}
```

정말 정말 복잡합니다. (null check 는 모두 제외 하였음) 만약 cutomerId 가 int 에서 long 으로 바뀐다면 코드의 변경은 쉽지 않을 것입니다.  

## Apache Avro

```json
{
 "namespace": "customerManagement.avro",
 "type": "record",
 "name": "Customer",
 "fields": [
	 {"name": "id", "type": "int"},
	 {"name": "name", "type": ["null", "string"]}
 ]
}
```
Avro 는 Json 으로 직렬화 방식을 표현합니다. 만약 데이터가 변경되면 json 값을 수정하면 됩니다. json Schema 파일을 이용해 직렬화 방식을 표현하는 것은 상당히 좋은 전략으로 보이지만 다음을 고려 해야 합니다.

> [!warning] two caveats
> * 데이터를 write 하는데 사용한 schema 와 read 하는데 사용한 schema는 호환 가능해야 합니다.
> * 역직렬화 과정에서 데이터를 write 하는데 사용한 schema 를 이용할 수 있어야 합니다.
> 

## Apache Avro - `Schema Registry` Pattern

schema registry 는 데이터를 읽는 측과 쓰는 측에서 필요한 schema 를 한곳에서 관리하기 위해 사용합니다. schema registry 는 Apache Kafka 의 프로젝트는 아니며 다양한 구현 프로젝트가 있습니다. 

Apache Avro 를 포함한 Schema Registry 패턴에 대한 이해는 이책의 범위를 넘어섭니다. 등장 배경 및 간략한 예제는 [Baeldung > Guide to Apache Avro](https://www.baeldung.com/java-apache-avro)을 참고 합시다.

## Partitions

이전 예제에서 `ProducerRecord` 객체는 topic 의 이름, key, value 를 포함하여 생성되었습니다. 하지만 Kafka 의 `ProducerRecord` 는 key 없이 topic 과 value 만으로 생성 될 수 있습니다. 이때 key 의 기본 값은 null 입니다.

key 는 두가지 목적을 위해 사용합니다.
- message 를 위한 추가적인 정보를 제공합니다.
- topic 의 어떤 파티션에 메시지를 저장할 지 결정합니다.

동일한 key 를 가진 메시지는 Kafka 의 독자적인 해싱 알고리즘을 통해 항상 동일한 partition 으로 할당됩니다.
	- 하지만 파티션의 개수가 변경된다면 이는 보장할 수 없습니다.
key 값이 null 인 메시지는 round-robin 으로 균등하게 할당됩니다.

