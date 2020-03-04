1.消息队列中队列的创建问题 







2.消息队列中消费者无法监听到消息队列的问题

可能出现的原因

> (1)某个消费者线程因内存溢出而挂掉，造成对应的队列没有消费者，消息在MQ Server堆积，而系统缺少对该类异常的监控，无法及时有效的进行处理。
> (2)在一些业务场景，消息的消费速度远低于生产速度，造成大量消息堆积在MQ Server，系统没有提供相应的机制来动态扩展消息的消费速度。
> (3)联调测试时，某些场景需要停止（或重启）客户端对消息队列的监听，系统没有处理这类操作的功能。



解决的方案

2.1 实施消费者的异常监控

spring通过发布事件的方式，可以通知观察者（即事件监听器）消费者的一些行为，消费者相关的事件如下所示。

* AsyncConsumerStartedEvent：An event that is published whenever a new consumer is started.

  `监听者启用的事件`

* AsyncConsumerRestartedEvent：An event that is published whenever a consumer is restarted.

  `监听者重启的事件`

* ListenerContainerConsumerFailedEvent：Published when a listener consumer fails.

  `监听者失败的事件`

* AsyncConsumerStoppedEvent：An event that is published whenever a consumer is stopped (and not restarted).

  `监听者停止的事件`

**基于事件机制，可以通过监听事件ListenerContainerConsumerFailedEvent，当有消费者发生致命错误时，重新创建消费者消费消息，并发送告警信息给相关责任人。具体实现如下：**

```java
import java.util.Arrays;

import org.springframework.amqp.rabbit.listener.ListenerContainerConsumerFailedEvent;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;

import lombok.extern.slf4j.Slf4j;

/**
 * MQ消费者失败事件监听器
 * @author wxyh
 * @date 2018/04/02
 */
@Slf4j
@Component
public class ListenerContainerConsumerFailedEventListener implements ApplicationListener<ListenerContainerConsumerFailedEvent> {

    @Override
    public void onApplicationEvent(ListenerContainerConsumerFailedEvent event) {
        log.error("消费者失败事件发生：{}", event);

        if (event.isFatal()) {
            log.error(String.format("Stopping container from aborted consumer. Reason::%s.",
                    event.getReason()), event.getThrowable());
            //消费者监听的方法
            SimpleMessageListenerContainer container = (SimpleMessageListenerContainer) event.getSource();
            String queueNames = Arrays.toString(container.getQueueNames());
            // 重启
            try {
                restart(container);
                log.info("重启队列%s的监听成功！", queueNames);
            } catch (Exception e) {
                log.error(String.format("重启队列%s的监听失败！", queueNames), e);
            }

            // TODO 告警，包含队列信息，监听断开原因，断开时异常信息，重启是否成功等...

        }
    }

    /**
     * 重启监听
     * @param container
     * @return
     */
    private void restart(SimpleMessageListenerContainer container) {
        // 暂停30s
        try {
            Thread.sleep(30000);
        } catch (Exception e) {
            log.error(e.getMessage());
        }

        Assert.state(!container.isRunning(), String.format("监听容器%s正在运行！", container));
        container.start();
    }

}
```



2.3动态修改消费者的消费数量

消费者监听队列需要与MQServer保持长连接，平时数据压力不大时，多个消费者同时监听一个队列，对系统资源也是一种浪费。动态修改消费者数量可以简单可靠的解决MQServer数据堆积问题。

```java
// org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer源码片段
public class SimpleMessageListenerContainer extends AbstractMessageListenerContainer
        implements ApplicationEventPublisherAware {
    // 并发消费者数量默认为1
    private volatile int concurrentConsumers = 1;

    public void setConcurrentConsumers(final int concurrentConsumers) {
        // 动态增加或消费消费者，do other operation
    }

    // ......
}
```

**从源码可知，`volatile`变量`concurrentConsumers`，可以保证所有线程对它的可见性，因此可以通过修改该字段来改变某个队列的消费者数量，达到动态改变消息消费速度的目的，以应对消息堆积的场景。**



2.4停止/重启对消息队列的监听

>基于注解方式监听队列消息的核心类有如下几个。
>org.springframework.amqp.rabbit.annotation.RabbitListenerAnnotationBeanPostProcessor：
>负责发现所有bean中@RabbitListener的方法，并将其封装为MethodRabbitListenerEndpoint对象。
>org.springframework.amqp.rabbit.listener.MethodRabbitListenerEndpoint：
>每个@RabbitListener注解的方法对应一个该类型的对象，提供消息的处理方法。
>org.springframework.amqp.rabbit.listener.RabbitListenerEndpointRegistrar：
>注册endpoint的工具bean。
>org.springframework.amqp.rabbit.listener.RabbitListenerEndpointRegistry：
>负责创建MessageListenerContainer实例，并管理所有监听容器的启动与停止等。



> 应用启动时，创建消息监听容器MessageListenerContainer的流程如下：
> （1）RabbitListenerAnnotationBeanPostProcessor会获取每个bean的@RabbitListener注解的方法；
> （2）根据bean和@RabbitListener属性值创建一个MethodRabbitListenerEndpoint类型的对象endpoint；
> （3）从容器中获取@RabbitListener指定的containerFactory bean；
> （4）根据endpoint和containerFactory 创建MessageListenerContainer实例，并保存在listenerContainers中。
>
> RabbitListenerEndpointRegistry实例创建的MessageListenerContainer实例在其整个生命周期都是有状态的。
> SimpleMessageListenerContainer的start()方法，负责创建消费者并启动对消息队列的监听。
> SimpleMessageListenerContainer的stop()方法，负责销毁消费者并停止对消息队列的监听。
> 因此，可以通过这两个方法达到对消息队列监听的停止与重启。
>

代码实现

3.1MQClientMonitor

MQ客户端监控器，提供了2.3与2.4的具体实现代码。

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

import org.apache.commons.lang3.StringUtils;
import org.springframework.amqp.rabbit.listener.RabbitListenerEndpointRegistry;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;

import lombok.Data;

/**
 * MQ客户端监控器
 * @author wxyh
 * @date 2018/04/04
 */
@Component
public class MQClientMonitor {

    private static final String CONTAINER_NOT_EXISTS = "消息队列%s对应的监听容器不存在！";

    @Autowired
    private RabbitListenerEndpointRegistry registry;

    /**
     * queue2ContainerAllMap初始化标识
     */
    private volatile boolean hasInit = false;

    /**
     * 所有的队列监听容器MAP
     */
    private final Map<String, SimpleMessageListenerContainer> allQueue2ContainerMap = new ConcurrentHashMap<>();

    /**
     * 重置消息队列并发消费者数量
     * @param queueName
     * @param concurrentConsumers must greater than zero
     * @return
     */
    public boolean resetQueueConcurrentConsumers(String queueName, int concurrentConsumers) {
        Assert.state(concurrentConsumers > 0, "参数 'concurrentConsumers' 必须大于0.");
        SimpleMessageListenerContainer container = findContainerByQueueName(queueName);
        if (container.isActive() && container.isRunning()) {
            container.setConcurrentConsumers(concurrentConsumers);
            return true;
        }
        return false;
    }


    /**
     * 重启对消息队列的监听
     * @param queueName
     * @return
     */
    public boolean restartMessageListener(String queueName) {
        SimpleMessageListenerContainer container = findContainerByQueueName(queueName);
        Assert.state(!container.isRunning(), String.format("消息队列%s对应的监听容器正在运行！", queueName));
        container.start();
        return true;
    }

    /**
     * 停止对消息队列的监听
     * @param queueName
     * @return
     */
    public boolean stopMessageListener(String queueName) {
        SimpleMessageListenerContainer container = findContainerByQueueName(queueName);
        Assert.state(container.isRunning(), String.format("消息队列%s对应的监听容器未运行！", queueName));
        container.stop();
        return true;
    }

    /**
     * 统计所有消息队列详情
     * @return
     */
    public List<MessageQueueDatail> statAllMessageQueueDetail() {
        List<MessageQueueDatail> queueDetailList = new ArrayList<>();
        getQueue2ContainerAllMap().entrySet().forEach(entry -> {
            String queueName = entry.getKey();
            SimpleMessageListenerContainer container = entry.getValue();
            MessageQueueDatail deatil = new MessageQueueDatail(queueName, container);
            queueDetailList.add(deatil);
        });

        return queueDetailList;
    }

    /**
     * 根据队列名查找消息监听容器
     * @param queueName
     * @return
     */
    private SimpleMessageListenerContainer findContainerByQueueName(String queueName) {
        String key = StringUtils.trim(queueName);
        SimpleMessageListenerContainer container = getQueue2ContainerAllMap().get(key);
        Assert.notNull(container, String.format(CONTAINER_NOT_EXISTS, key));
        return container;
    }

    private Map<String, SimpleMessageListenerContainer> getQueue2ContainerAllMap() {
        if (!hasInit) {
            synchronized (allQueue2ContainerMap) {
                if (!hasInit) {
                    registry.getListenerContainers().forEach(container -> {
                        SimpleMessageListenerContainer simpleContainer = (SimpleMessageListenerContainer) container;
                        Arrays.stream(simpleContainer.getQueueNames()).forEach(queueName ->
                        allQueue2ContainerMap.putIfAbsent(StringUtils.trim(queueName), simpleContainer));
                    });
                    hasInit = true;
                }
            }
        }
        return allQueue2ContainerMap;
    }


    /**
     * 消息队列详情
     * @author liuzhe
     * @date 2018/04/04
     */
    @Data
    public static final class MessageQueueDatail {
        /**
         * 队列名称
         */
        private String queueName;

        /**
         * 监听容器标识
         */
        private String containerIdentity;

        /**
         * 监听是否有效
         */
        private boolean activeContainer;

        /**
         * 是否正在监听
         */
        private boolean running;

        /**
         * 活动消费者数量
         */
        private int activeConsumerCount;

        public MessageQueueDatail(String queueName, SimpleMessageListenerContainer container) {
            this.queueName = queueName;
            this.running = container.isRunning();
            this.activeContainer = container.isActive();
            this.activeConsumerCount = container.getActiveConsumerCount();
            this.containerIdentity = "Container@" + ObjectUtils.getIdentityHexString(container);
        }

    }

}
```



## 3.2MQManageController

MQ管理控制器，提供相应功能的入口，具体见下图。

![这里写图片描述](RabbitMQ%E9%97%AE%E9%A2%98.assets/20180405125430804.png)

```java
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.wxyh.springbootproject.config.mq.MQClientMonitor;
import com.wxyh.springbootproject.config.mq.MQClientMonitor.MessageQueueDatail;

import io.swagger.annotations.ApiOperation;

/**
 * MQ管理控制器
 * @author wxyh
 * @date 2018/04/04
 */
@RestController
@RequestMapping("/mqManage")
public class MQManageController {

    @Autowired(required = false)
    private MQClientMonitor mqClientMonitor;

    /**
     * 重置指定队列消费者数量
     * @param queueName
     * @param concurrentConsumers
     * @return
     */
    @ApiOperation("重置指定队列消费者数量")
    @GetMapping("resetConcurrentConsumers")
    public boolean resetConcurrentConsumers(String queueName, int concurrentConsumers) {
        return mqClientMonitor.resetQueueConcurrentConsumers(queueName, concurrentConsumers);
    }

    /**
     * 重启对消息队列的监听
     * @param queueName
     * @return
     */
    @ApiOperation("重启对消息队列的监听")
    @GetMapping("restartMessageListener")
    public boolean restartMessageListener(String queueName) {
        return mqClientMonitor.restartMessageListener(queueName);
    }

    /**
     * 停止对消息队列的监听
     * @param queueName
     * @return
     */
    @ApiOperation("停止对消息队列的监听")
    @GetMapping("stopMessageListener")
    public boolean stopMessageListener(String queueName) {
        return mqClientMonitor.stopMessageListener(queueName);
    }

    /**
     * 获取所有消息队列对应的消费者
     * @return
     */
    @ApiOperation("统计所有消息队列详情")
    @GetMapping("statAllMessageQueueDetail")
    public List<MessageQueueDatail> statAllMessageQueueDetail() {
        return mqClientMonitor.statAllMessageQueueDetail();
    }

}
```





[详情](https://blog.csdn.net/u011424653/article/details/79824538)



### 生产者

```java
/**
 * 商品视频MQ的生产者
 */
@Component
@Slf4j
public class GoodsVideoRabbitProducer {

    @Autowired
    private AmqpTemplate amqpTemplate;
    /**
     * 监听服务端给我们的返回确认的请求 , 如果消息到了交换机就返回true
     *
     * @ack true false
     */
    private final RabbitTemplate.ConfirmCallback confirmCallback = (correlationData, ack, cause) -> {
        if (!ack) {
            //设置消息没有到达交换机的补偿
            log.error("消息发送失败: 消息没有到达交换机:信息为 error={}", cause);
        }
    };

    /**
     * 监听对不可达的消息进行后续处理;
     * 不可达消息：指定的路由key路由不到。
     */
    private final RabbitTemplate.ReturnCallback returnCallback = (message, replyCode, replyText, exchange, routingKey) -> {
        log.info("消息发送失败: 交换机未能找到对应路由键的队列");
        log.info("发送失败的{交换机,路由键}信息为:", exchange, routingKey);
        log.error("发送失败的回应内容为: error={}", replyText);
    };


    /**
     * 审核的队列和路由键
     *
     * @param vo
     */
    public void sendGoodsVideoAuditMQ(AuditGoodsVideoVo vo) {
        //(交换机,路由键,消息体)
        Map<String, Object> map = new HashMap<>();
        map.put("GoodsAudit", vo);
        map.put("remark", "一个单独的消息,一个消息是一个实体");
        try {
            log.info("同步商品视频数据推送到 upms_goodsvideo 消息队列");
            amqpTemplate.convertAndSend(RabbitExchange.EXCHANGE_UPMS_GOODSVIDEO, RabbitRoutingKey.ROUTING_KEY_UPMS_GOODSVIDEO_KEY, JSONObject.toJSONString(map));
        } catch (Exception e) {
            log.error("同步商品视频数据推送失败 错误的队列:<upms_goodsvideo> 信息为: error={}", e);
        }
    }


}
```



