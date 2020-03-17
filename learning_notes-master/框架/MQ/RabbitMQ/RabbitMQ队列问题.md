# RabbitMQ 自动创建队列/交换器/绑定

\#自动创建队列，什么玩意儿？

> 在没有使用Spring AMQP之前，如果我们使用的是官方的Java客户端，我们需要自己手动调用创建channel，手动调用`channel.queueDeclare()`方法来创建队列。

> 如果使用Spring AMQP来操作RabbitMQ的时候,我们会做些什么呢？在配置文件里配置`Queue`, `XXExchange`, `Binding`等信息。配置完成之后我们启动Spring容器，容器启动后，我们定义的队列也好，交换器也好，都会自动为我们创建。这就是Spring AMQP封装的自动创建队列。

> 如果我们要自己实现配置化自动创建队列，交换器如何实现呢？下面我们会细细将来。

\#什么是RabbitAdmin？

> RabbitAdmin是Spring AMQP封装的一个对象。 在使用Spring AMQP的时候我们只是配置一个RabbitAdmin对象交给Spring容器做管理。

\#Spring 自动创建队列原理

> 我们在使用Spring AMQP的时候往往就是声明连接工厂，Queue， Exchange，Binding。

```
@Bean



public Queue mailQueue() {



    Map<String, Object> map = new HashMap<String, Object>();



    map.put("x-dead-letter-exchange", "dead_letter_exchange");//设置死信交换机



    map.put("x-dead-letter-routing-key", "mail_queue_fail");//设置死信routingKey



    Queue queue = new Queue("mailQueue",true, false, false, map);



    return queue;



}



 



@Bean



public DirectExchange mailExchange() {



    return new DirectExchange("mailExchange", true, false);



}



 



@Bean



public Binding mailBinding() {



    return BindingBuilder.bind(mailQueue()).to(mailExchange())



            .with(mailRoutingKey);



}
```

> 我们只是声明了这些东西，它们怎么就创建了呢，要知道我们没有显式调用`channel.queueDeclare()`啊。

> 接下来我们来看下为什么有了RabbitAdmin对象就可以自动创建队列。

\####RabbitAdmin的初始化方法`initialize()`

> 我们看了RabbitAdmin的`initialize()`方法的源码就知道队列是怎么自动创建的了。

```
	if (this.applicationContext == null) {



		if (this.logger.isDebugEnabled()) {



			this.logger



					.debug("no ApplicationContext has been set, cannot auto-declare Exchanges, Queues, and Bindings");



		}



		return;



	}



 



	this.logger.debug("Initializing declarations");



	Collection<Exchange> contextExchanges = new LinkedList<Exchange>(



			this.applicationContext.getBeansOfType(Exchange.class).values());



	Collection<Queue> contextQueues = new LinkedList<Queue>(



			this.applicationContext.getBeansOfType(Queue.class).values());



	Collection<Binding> contextBindings = new LinkedList<Binding>(



			this.applicationContext.getBeansOfType(Binding.class).values());
```

> 通过代码片段，我们可以看到，RabbitAdmin初始化的时候会从spring容器里取出所有的交换器bean, 队列bean, Binding Bean; 取出之后干什么呢，当然是创建了，下面来看下创建代码

```
	this.rabbitTemplate.execute(new ChannelCallback<Object>() {



		@Override



		public Object doInRabbit(Channel channel) throws Exception {



			declareExchanges(channel, exchanges.toArray(new Exchange[exchanges.size()]));



			declareQueues(channel, queues.toArray(new Queue[queues.size()]));



			declareBindings(channel, bindings.toArray(new Binding[bindings.size()]));



			return null;



		}



	});
```

> 到此，我们通过Spring配置的Queue对象也好，XXExchange对象也好，Binding也罢，都会在这里被创建。

\####什么时候调用的`initialize()`

> 虽然我们知道了是在`initialize()`方法中实现的自动创建队列等信息，但是这个什么时候被调用的？ RabbitAdmin实现InitializingBean接口，如果你对Spring非常了解的话，立马可以想到`afterPropertiesSet()`，有了它当然可以在bean创建初始化的时候执行某些方法了，`initialize()`方法就是在这个时候被调用的。

# 生产者端需要配置RabbitAdmin

> 生产者端不像消费端的`SimpleMessageListenerContainer`一样，可以自动初始化RabbitAdmin对象，需要我们手动来创建。

\####配置RabbitAdmin

```
@Bean



public RabbitAdmin rabbitAdmin(ConnectionFactory defaultConnectionFactory){



	return new RabbitAdmin(defaultConnectionFactory);



}
```

# 消费端会不用配置RabbitAdmin, Why?

\####消费端核心配置SimpleMessageListenerContainer类

```
@Bean



public SimpleMessageListenerContainer messageListenerContainer() {



    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();



    container.setConnectionFactory(connectionFactory());



    ...



    return container;



}
```

\####SimpleMessageListenerContainer的doStart()方法

> 我们来看下`SimpleMessageListenerContainer`类的初始化配置，也就是`doStart()`方法，因为代码太长，这里就不粘贴全部了,看下面的源码

```
...



if (this.rabbitAdmin == null && this.getApplicationContext() != null) {



	Map<String, RabbitAdmin> admins = this.getApplicationContext().getBeansOfType(RabbitAdmin.class);



	if (!admins.isEmpty()) {



		this.rabbitAdmin = admins.values().iterator().next();



	}



}



if (this.rabbitAdmin == null && this.autoDeclare) {



	RabbitAdmin rabbitAdmin = new RabbitAdmin(this.getConnectionFactory());



	rabbitAdmin.setApplicationContext(this.getApplicationContext());



	this.rabbitAdmin = rabbitAdmin;



}



...
```

> 如果没有指定rabbitAdmin但是autoDeclare为true，那么spring就会创建一个RabbitAdmin对象。有了RabbitAdmin对象，Spring就会为我们创建队列了。