# [RabbitMQ整合Spring Booot【死信队列】](https://www.cnblogs.com/toov5/p/10288260.html)

Config:



```java
import java.util.HashMap;
import java.util.Map;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

//Fanout 类型 发布订阅模式
@Component
public class FanoutConfig {

    /**
     * 定义死信队列相关信息
     */
    public final static String deadQueueName = "dead_queue";
    public final static String deadRoutingKey = "dead_routing_key";
    public final static String deadExchangeName = "dead_exchange";
    /**
     * 死信队列 交换机标识符
     */
    public static final String DEAD_LETTER_QUEUE_KEY = "x-dead-letter-exchange";
    /**
     * 死信队列交换机绑定键标识符
     */
    public static final String DEAD_LETTER_ROUTING_KEY = "x-dead-letter-routing-key";

    // 邮件队列
    private String FANOUT_EMAIL_QUEUE = "fanout_email_queue";

    // 短信队列
    private String FANOUT_SMS_QUEUE = "fanout_sms_queue";
    // fanout 交换机
    private String EXCHANGE_NAME = "fanoutExchange";

    // 1.定义邮件队列
    @Bean
    public Queue fanOutEamilQueue() {
        // 将普通队列绑定到死信队列交换机上
        Map<String, Object> args = new HashMap<>(2);
        args.put(DEAD_LETTER_QUEUE_KEY, deadExchangeName);
        args.put(DEAD_LETTER_ROUTING_KEY, deadRoutingKey);
        Queue queue = new Queue(FANOUT_EMAIL_QUEUE, true, false, false, args);
        return queue;
    }

    // 2.定义短信队列
    @Bean
    public Queue fanOutSmsQueue() {
        return new Queue(FANOUT_SMS_QUEUE);
    }

    // 2.定义交换机
    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange(EXCHANGE_NAME);
    }

    // 3.队列与交换机绑定邮件队列
    @Bean
    Binding bindingExchangeEamil(Queue fanOutEamilQueue, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanOutEamilQueue).to(fanoutExchange);
    }

    // 4.队列与交换机绑定短信队列
    @Bean
    Binding bindingExchangeSms(Queue fanOutSmsQueue, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanOutSmsQueue).to(fanoutExchange);
    }

    /**
     * 创建配置死信邮件队列
     * 
     * @return
     */
    @Bean
    public Queue deadQueue() {
        Queue queue = new Queue(deadQueueName, true);
        return queue;
    }
   /*
    * 创建死信交换机
    */
    @Bean
    public DirectExchange deadExchange() {
        return new DirectExchange(deadExchangeName);
    }
   /*
    * 死信队列与死信交换机绑定
    */
    @Bean
    public Binding bindingDeadExchange(Queue deadQueue, DirectExchange deadExchange) {
        return BindingBuilder.bind(deadQueue).to(deadExchange).with(deadRoutingKey);
    }

}
```



 

 生产者 timestamp 设置为0 

`设置消息头`

```java
package com.itmayiedu.rabbitmq;

import java.util.UUID;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageBuilder;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.alibaba.fastjson.JSONObject;

@Component
public class FanoutProducer {
    @Autowired
    private AmqpTemplate amqpTemplate;

    public void send(String queueName) {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("email", "xx@163.com");
        jsonObject.put("timestamp", 0);
        String jsonString = jsonObject.toJSONString();
        System.out.println("jsonString:" + jsonString);
        // 设置消息唯一id 保证每次重试消息id唯一 
        Message message = MessageBuilder.withBody(jsonString.getBytes())
                .setContentType(MessageProperties.CONTENT_TYPE_JSON).setContentEncoding("utf-8")
                .setMessageId(UUID.randomUUID() + "").build(); //消息id设置在请求头里面 用UUID做全局ID 
        amqpTemplate.convertAndSend(queueName, message);
    }
}
```



 

 ![img](RabbitMQ%E6%AD%BB%E4%BF%A1%E9%98%9F%E5%88%972.assets/1179709-20190118145756500-1253491390.png)

 此时的消费者：



```java
@RabbitListener(queues = "fanout_email_queue")
    public void process(Message message, @Headers Map<String, Object> headers, Channel channel) throws Exception {
        // 获取消息Id
        String messageId = message.getMessageProperties().getMessageId();
        String msg = new String(message.getBody(), "UTF-8");
        System.out.println("邮件消费者获取生产者消息msg:"+msg+",消息id"+messageId);
        
        JSONObject jsonObject = JSONObject.parseObject(msg);
        Integer timestamp = jsonObject.getInteger("timestamp");
        
        try {
            int result  = 1/timestamp;
            System.out.println("result"+result);
            // // 手动ack
            Long deliveryTag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
            // 手动签收
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            //拒绝消费消息（丢失消息） 给死信队列
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }
        
        System.out.println("执行结束....");
    }
```

[![复制代码](RabbitMQ%E6%AD%BB%E4%BF%A1%E9%98%9F%E5%88%972.assets/copycode.gif)](javascript:void(0);)

 

 异常状况：

![img](RabbitMQ%E6%AD%BB%E4%BF%A1%E9%98%9F%E5%88%972.assets/1179709-20190118151337177-1519715031.png)

 

 

添加死信队列的消费者，并启动后：

 



```java
package com.itmayiedu.rabbitmq;

import java.util.Map;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.stereotype.Component;

import com.rabbitmq.client.Channel;

//死信队列
@Component
public class FanoutDeadEamilConsumer {    
    
    @RabbitListener(queues = "dead_queue")
    public void process(Message message, @Headers Map<String, Object> headers, Channel channel) throws Exception {
        // 获取消息Id
        String messageId = message.getMessageProperties().getMessageId();
        String msg = new String(message.getBody(), "UTF-8");
        System.out.println("死信邮件消费者获取生产者消息msg:"+msg+",消息id"+messageId);
        // // 手动ack
        Long deliveryTag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
       // 手动签收
       channel.basicAck(deliveryTag, false);
        
        System.out.println("执行结束....");
    }    
    
}
```



 

![img](RabbitMQ%E6%AD%BB%E4%BF%A1%E9%98%9F%E5%88%972.assets/1179709-20190118151823094-127493502.png)

 

