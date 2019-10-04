---
title: rabbitmq 설정
date: 2017-12-05
categories: rabbitmq
---

rabbitmq 설정

큐와 exchange 등은 rabbitmq 관리페이지에서 직접 생성한다면, 기본은 아래와 같이 ConnectionFactory / AmqpTemplate / ListenerContainerFactory 만 설정하면 됨.

```java

@Bean
public ConnectionFactory defaultConnectionFactory() {
   CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
   connectionFactory.setAddresses(coreProperties.getProperty("amqp.addresses"));
   connectionFactory.setUsername(coreProperties.getProperty("amqp.username"));
   connectionFactory.setPassword(coreProperties.getProperty("amqp.password"));
   connectionFactory.setVirtualHost(coreProperties.getProperty("amqp.vhost"));

   connectionFactory.setCacheMode(CachingConnectionFactory.CacheMode.CHANNEL);
   connectionFactory.setChannelCacheSize(50); //TODO 조절 필요
   //    connectionFactory.setConnectionNameStrategy(cf -> "INTERWORK_CONNECTION"); //TODO 얘는 어디에서 쓰는지 확인 필요
   return connectionFactory;
}

@Bean
public MessageConverter jsonMessageConverter() {
   Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter(JsonConverters.nonstrict().getObjectMapper());
   return jackson2JsonMessageConverter;
}

@Bean
public AmqpTemplate rabbitTemplate() {
   RabbitTemplate template = new RabbitTemplate(defaultConnectionFactory());
   template.setExchange("interwork.external.direct”); // 요게 exchange 이름..
   return template;
}

@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
   SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
   factory.setConnectionFactory(defaultConnectionFactory());
   factory.setConcurrentConsumers(3); // 리스너 갯수
   factory.setMaxConcurrentConsumers(5); // 최대 리스너 갯수
   factory.setMessageConverter(jsonMessageConverter());
   return factory;
}
```

여기에 retry 시도를 위해 retry 설정을 추가. 최대 3번까지 리트라이를 하고, 그래도 실패하면 Recoverer가 실행되도록 하는 설정.  
backOffPolicy는 backOff 정책을 설정하는 것으로, 대충 초기 인터벌 값에서 멀티플라이어 값을 곱한 시간만큼 대기하다가 재시도,  
그래도 실패하면 그 전 시도한 인터벌 값에서 다시 멀티플라이어를 곱한 뒤 재시도.. 정해진 재시도 횟수가 될때까지 재시도함.  
정해진 횟수만큼 재시도해도 실패했을 경우 recoverer 동작..  
RepublishMessageRecoverer 와 RejectAndDontRequeueRecoverer 가 있는데,  
Republish~~ 는 이름 그대로 queue에 다시 쌓음.. Reject~~ 역시 이름그대로 에러났어도 그냥 버림.  
Reject~~ recoverer를 써야하는데, 로그를 추가하고 싶어서 Reject~~를 상속받은 WriteLogRecover를 만듬.  
이렇게 하면 3회 재시도 후 body 의 내용을 로그로 남긴 후 큐에서 버리게 됨.

```java
@Bean
public RetryOperationsInterceptor retryInterceptor() {
   ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
   backOffPolicy.setInitialInterval(1_000);
   backOffPolicy.setMultiplier(3.0);
   backOffPolicy.setMaxInterval(5_000);

   return RetryInterceptorBuilder.stateless()
         .maxAttempts(3)
         .backOffPolicy(backOffPolicy)
         .recoverer(new WriteLogRecover())
         .build();
}
```

Recoverer
```java
public class WriteLogRecover extends RejectAndDontRequeueRecoverer {

   private static final Logger LOGGER = LoggerFactory.getLogger(WriteLogRecover.class);

   @Override
   public void recover(Message message, Throwable cause) {
      String messsageString = null;
      try {
         messsageString = new String(message.getBody(), "UTF-8");

      } catch (Exception e) {
         LOGGER.warn("WriteLogRecover messageString convert error");

      } finally {
         LOGGER.error("WriteLogRecover retry fail.. value : " + messsageString);
      }

      super.recover(message, cause);
   }
}
```

Listener
```java
@Component
public class SnapshotListener {
   private static final Logger LOGGER = LoggerFactory.getLogger(SnapshotListener.class);

   @Autowired
   @Qualifier("interworkSnapshotBucketRepository")
   private InterworkCouchbaseRepository couchbaseRepository;

   @Autowired
   private SnapshotService snapshotService;

   @RabbitListener(bindings =
   @QueueBinding(value = @Queue(value = "interwork.external.snapshot"),
         exchange = @Exchange(value = "interwork.external.direct"),
         key = "interwork.couchbase.addLog"))
   public void saveSnapshot(SnapshotDocument snapshotDocument) {
      LOGGER.info("@@@@@ SnapshotListener start!! document id : " + snapshotDocument.getId());

      String documentId = snapshotDocument.getId();
      if (documentId == null) {
         return;
      }

      SnapshotDocument origin = snapshotService.get(documentId);

      if (origin == null) {
         origin = snapshotDocument;
         couchbaseRepository.save(snapshotDocument);
      } else {
         List<SnapshotElement> originElementList = origin.getSnapshotElementList();

         // snapshotDocument는 1 document 1 element.. list 통으로 add..
         originElementList.addAll(snapshotDocument.getSnapshotElementList());
         couchbaseRepository.save(origin);
      }

      LOGGER.info("@@@@@ SnapshotListener end!! document id : " + origin.getId());

   }
}
```

queue 및 exchange 등도 서버에서  operation 하고싶다면 RabbitAdmin 설정이 필요함.  
아래와 같이 admin, exchange, queue, binding 설정을 해주면 됨. 여기서 바인딩이 여러개 필요하다면 List<Binging> 을 리턴하도록 구성…. 

```java

   @Bean
   public RabbitAdmin admin(ConnectionFactory cf) {
      return new RabbitAdmin(cf);
   }

   @Bean
   public Queue queue() {
      Map<String, Object> arguments = new HashMap<>();
      arguments.put("x-ha-policy", "all");
      return new Queue("interwork.mq.test", true, false, false, arguments);
   }

   @Bean
   public DirectExchange exchange() {
      return new DirectExchange("interwork.ex.direct.test");
   }

   @Bean
   public Binding binding() {
      return BindingBuilder.bind(queue()).to(exchange()).with("interwork.rk.test");
   }

   @Bean
   public AmqpTemplate rabbitTemplate() {
      RabbitTemplate template = new RabbitTemplate(defaultConnectionFactory());
      template.setMessageConverter(jsonMessageConverter());
//    template.setExchange(deployEnvironment.getProperty("amqp.exchange.direct"));
      template.setExchange("interwork.ex.direct.test");
      return template;
   }
```
