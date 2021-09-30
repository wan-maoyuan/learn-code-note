## 现在在xml里面导入包
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
    <version>3.0.1.RELEASE</version>
</dependency>
```

## 在配置文件配置kafka信息
```yml
    stream:
      kafka:
        binder:
          brokers: 192.168.3.3:9092         #Kafka的消息中间件服务器
          #          zk-nodes: localhost:2181        #Zookeeper的节点，如果集群，后面加,号分隔
          auto-create-topics: true        #如果设置为false,就不会自动创建Topic 有可能你Topic还没创建就直接调用了。
          auto-add-partitions: true
      bindings:
        #接受到的order放入和取出
        output:
          destination: ReceiveOrder    #消息发往的目的地
#          content-type: text/plain    #消息发送的格式，接收端不用指定格式，但是发送端要
          content-type: application/json    #消息发送的格式，接收端不用指定格式，但是发送端要
          producer:
            partitionCount: 1
        input:
          destination: ReceiveOrder
          group: ReceiveOrder-group
          consumer:
            partitioned: true   #开启分区

        #接收到的result放入和取出
        resultOutput:
          destination: ReceiveResult
          content-type: application/json
          producer:
            partitionCount: 1

        resultInput:
          destination: ReceiveResult
          group: ReceiveResult-group
          consumer:
            partitioned: true   #开启分区
```

## 配置kafka的消息管道
```java 
/**
 * 第三方测试结果的输入输出管道
 * @author WMY
 */
public interface ResultChannel {

    String INPUT = "resultInput";
    String OUTPUT = "resultOutput";

    /**
     * 结果输入管道
     * @return
     */
    @Input(ResultChannel.INPUT)
    SubscribableChannel input();

    /**
     * 结果输出管道
     * @return
     */
    @Output(ResultChannel.OUTPUT)
    MessageChannel output();

}
```

## 往kafka消息管道中发送数据
```java 
/**
 * 将医嘱信息发送到kafka中
 * @author big_john
 */
@EnableBinding(Source.class)
public class SendOrderKafka {

    @Resource
    private Source source;

    public boolean send(InfoFromLis order) {
        try {
            return source.output().send(MessageBuilder.withPayload(order).build());
        }catch (Exception e){
            return false;
        }
    }

}
```

## 从kafka消息管道中获取数据
```java
/**
 * 从kafka中获取医嘱信息进行信息处理
 * @author WMY
 */
@Slf4j
@EnableBinding(Sink.class)
public class ReceiveOrderKafka {

    @Resource
    private SampleInfoMapper sampleInfoMapper;
    @Resource
    private ProgItemMapper progItemMapper;

    @StreamListener(Sink.INPUT)
    public void receive(InfoFromLis order) {
        boolean flag = saveOrder(order);
        if (flag) {
            log.info(order.getSampleId() + "保存成功");
        }else {
            log.error(order.getSampleId() + "保存失败");
        }
    }
}
```

