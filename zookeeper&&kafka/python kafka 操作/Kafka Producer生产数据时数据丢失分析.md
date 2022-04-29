# Kafka Producer生产数据时数据丢失分析

来源：[(170条消息) Kafka Producer生产数据时数据丢失分析_瓜牛呱呱的博客-CSDN博客_producer.flush](https://blog.csdn.net/Lin_wj1995/article/details/80264599)

> 今天在测试 Storm 程序过程中，想通过运行在 idea 的 Kafka Producer 生产一条数据来验证一下 Storm 程序，发现居然没有成功将数据生产到 Kafka 集群中，于是进行了一番测试，最终找到了原因！
> 注：下面程序测试中使用的 kafka 的版本为 0.10.2.0，zookeeper 的版本为 3.4.5

### **一、情景再现**



在 linux 中运行如下命令来监控是否有数据生产到 kafka 中：

> kafka-console-consumer --zookeeper localhost:2181 --topic test

生产一条数据到Kafka中，样例代码如下：

```java
package kafka;

import kafka.admin.AdminUtils;
import kafka.admin.RackAwareMode;
import kafka.utils.ZkUtils;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.security.JaasUtils;
import utils.PropertiesUtil;

import java.util.Properties;

/**
 * Kafka客户端操作工具类
 * 0.10.2版本
 * @author lwj
 * @date 2018/5/10
 */
public class KafkaProducerUtil {
    private static KafkaProducer<String, String> producer = null;
    private static Properties props = null;
    static {
        props = new Properties();
        props.put("bootstrap.servers", "XXX.XXX.XXX.XXX:XX");
        props.put("acks", "all");
        props.put("retries", "0");
        props.put("batch.size", "16384");
//        props.put("linger.ms", "1");
        props.put("buffer.memory", "33554432");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        producer = new KafkaProducer(props);
    }

    /**
     * 将消息生产到kafka集群
     * @param topic
     * @param message
     */
    public static void produce(String topic, String message){
        producer.send(new ProducerRecord<String, String>(topic, message));
    }


    public static void main(String[] args) {
        //生产多少条数据到Kafka（***）
        int num = 1
        for (int i = 0; i < num; i++) {
            KafkaProducerUtil.produce("test", "test"+i);
//            try {
//                Thread.sleep(1000);
//            } catch (InterruptedException e) {
//
//            }
        }
    }
}
```

发现并没有数据生产到 broker 中！！

### **二、原因分析**

如果将上面代码的 num 的值设置成 1000 的话，发现数据有生产到 Kafka 集群中！！
如果解开上面 Thread.sleep() 代码的话，发现也有数据有生产到 Kafka 集群中！！

所以，造成上面生产不成功的原因就是虽然调用了 producer.send() ，但是数据还没来得及生产到 Kafka 集群 主程序就挂掉了，于是数据就没有生产到 Kafka 集群中了~~

也就是说虽然 Kafka官网给出的文档中 Produer 的 linger.ms 参数 默认是 0，但是真实情况中 producer.send() 方法 “生效” 是有些许延迟的！

### **三、解决方法**

如果对性能要求不高的话，可以再 `producer.send()` 方法调用后再调用 `producer.flush()` 方法，该方法会将数据全部生产到Kafka，否则就会阻塞。对于 `producer.flush()` 方法，源码原话如下：

> "Flush any accumulated records form the producer. Blocks until all sends are complete."

但是这个方法有一点局限性，就是对性能的影响有点大，这个是要注意的地方~

如果对性能要求比较高，同时也想把数据确切的生产到集群的话，推荐将 linger.ms 参数设置一个比 0 大的值（默认是 0），batch.size 也可以设置一下（默认是16384），同时用 producer.send(ProducerRecord<K,V>, Callback) 来将数据生产到集群中，其中 Callback 匿名内部类中的 onCompletion() 方法用来处理 “确认生产到集群” 的逻辑~~