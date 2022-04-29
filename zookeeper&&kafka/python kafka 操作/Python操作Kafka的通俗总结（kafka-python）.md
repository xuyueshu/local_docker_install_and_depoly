# Python操作Kafka的通俗总结（kafka-python）

来源：[Python操作Kafka的通俗总结（kafka-python） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/279784873)

## 一、基本概念

- Topic：一组消息数据的标记符；
- Producer：生产者，用于生产数据，可将生产后的消息送入指定的Topic；
- Consumer：消费者，获取数据，可消费指定的Topic；
- Group：消费者组，同一个group可以有多个消费者，一条消息在一个group中，只会被一个消费者获取；
- Partition：分区，为了保证kafka的吞吐量，一个Topic可以设置多个分区。同一分区只能被一个消费者订阅。

## 二、本地安装与启动（基于Docker）

1. 下载zookeeper镜像与kafka镜像：

```
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka
```

2. 本地启动zookeeper

```
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper  
```

3. 本地启动kafka

```
docker run -d --name kafka --publish 9092:9092 --link zookeeper \
--env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
--env KAFKA_ADVERTISED_HOST_NAME=localhost \
--env KAFKA_ADVERTISED_PORT=9092 \
wurstmeister/kafka:latest 
```

注意：上述代码，将kafka启动在9092端口

4. 进入kafka bash

```
docker exec -it kafka bash
cd /opt/kafka/bin
```

5. 创建Topic，分区为2，Topic name为'kafka_demo'

```
kafka-topics.sh --create --zookeeper zookeeper:2181 \
--replication-factor 1 --partitions 2 --topic kafka_demo
```

6. 查看当前所有topic

```
kafka-topics.sh --zookeeper zookeeper:2181 --list
```

7. 安装kafka-python

```
pip install kafka-python
```

## 三、生产者（Producer）与消费者（Consumer）

生产者和消费者的简易Demo，这里一起演示：

```python
from kafka import KafkaProducer, KafkaConsumer
from kafka.errors import kafka_errors
import traceback
import json


def producer_demo():
    # 假设生产的消息为键值对（不是一定要键值对），且序列化方式为json
    producer = KafkaProducer(
        bootstrap_servers=['localhost:9092'], 
        key_serializer=lambda k: json.dumps(k).encode(),
        value_serializer=lambda v: json.dumps(v).encode())
    # 发送三条消息
    for i in range(0, 3):
        future = producer.send(
            'kafka_demo',
            key='count_num',  # 同一个key值，会被送至同一个分区
            value=str(i),
            partition=1)  # 向分区1发送消息
        print("send {}".format(str(i)))
        try:
            future.get(timeout=10) # 监控是否发送成功           
        except kafka_errors:  # 发送失败抛出kafka_errors
            traceback.format_exc()


def consumer_demo():
    consumer = KafkaConsumer(
        'kafka_demo', 
        bootstrap_servers=':9092',
        group_id='test'
    )
    for message in consumer:
        print("receive, key: {}, value: {}".format(
            json.loads(message.key.decode()),
            json.loads(message.value.decode())
            )
        )
```

这里建议起两个terminal，或者两个jupyter notebook页面来验证。

先执行消费者：

```
consumer_demo()
```

再执行生产者：

```
producer_demo()
```

会看到如下输出：

```
>>> producer_demo()
send 0
send 1
send 2
>>> consumer_demo()
receive, key: count_num, value: 0
receive, key: count_num, value: 1
receive, key: count_num, value: 2
```

## 四、消费者进阶操作

（1）初始化参数：

列举一些KafkaConsumer初始化时的重要参数：

- group_id

高并发量，则需要有多个消费者协作，消费进度，则由group_id统一。例如消费者A与消费者B，在初始化时使用同一个group_id。在进行消费时，一条消息被消费者A消费后，在kafka中会被标记，这条消息不会再被B消费（前提是A消费后正确commit）。

- key_deserializer， value_deserializer

与生产者中的参数一致，自动解析。

- auto_offset_reset

消费者启动的时刻，消息队列中或许已经有堆积的未消费消息，有时候需求是从上一次未消费的位置开始读（则该参数设置为earliest），有时候的需求为从当前时刻开始读之后产生的，之前产生的数据不再消费（则该参数设置为latest）。

- enable_auto_commit， auto_commit_interval_ms

是否自动commit，当前消费者消费完该数据后，需要commit，才可以将消费完的信息传回消息队列的控制中心。enable_auto_commit设置为True后，消费者将自动commit，并且两次commit的时间间隔为auto_commit_interval_ms。

（2）手动commit

```python
def consumer_demo():
    consumer = KafkaConsumer(
        'kafka_demo', 
        bootstrap_servers=':9092',
        group_id='test',
        enable_auto_commit=False
    )
    for message in consumer:
        print("receive, key: {}, value: {}".format(
            json.loads(message.key.decode()),
            json.loads(message.value.decode())
            )
        )
        consumer.commit()
```

（3）查看kafka堆积剩余量

在线环境中，需要保证消费者的消费速度大于生产者的生产速度，所以需要检测kafka中的剩余堆积量是在增加还是减小。可以用如下代码，观测队列消息剩余量：

```python
consumer = KafkaConsumer(topic, **kwargs)
partitions = [TopicPartition(topic, p) for p in consumer.partitions_for_topic(topic)]

print("start to cal offset:")

# total
toff = consumer.end_offsets(partitions)
toff = [(key.partition, toff[key]) for key in toff.keys()]
toff.sort()
print("total offset: {}".format(str(toff)))
    
# current
coff = [(x.partition, consumer.committed(x)) for x in partitions]
coff.sort()
print("current offset: {}".format(str(coff)))

# cal sum and left
toff_sum = sum([x[1] for x in toff])
cur_sum = sum([x[1] for x in coff if x[1] is not None])
left_sum = toff_sum - cur_sum
print("kafka left: {}".format(left_sum))
```