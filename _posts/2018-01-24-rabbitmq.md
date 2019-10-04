---
title: rabbitmq MessageConverter
date: 2018-01-24
categories: rabbitmq
---

rabbitmq의 messageConverer는 template과 listener factory에 각각 설정할 수 있으며, default는 SimpleMessageConverter.  
SimpleMessageConverter는 byte[] 로 string과 객체를 직렬화/역직렬화하는 기본 컨버터.  
SimpleMessageConverter 이외에 Jackson2JsonMessageConverter, ContentTypeDeletatingMessageConverter 등의 컨버터가 있다.  
ContentTypeDelegatingMessageConverter의 경우 이름에서 알 수 있듯, Message에 set된 contentType에 따라 알맞은 MessageConverter에 컨버팅을 위임한다.  


아래와 같이 사용 가능하다.  

```java

@Bean
public ContentTypeDelegatingMessageConverter contentTypeDelegatingMessageConverter() {
   ContentTypeDelegatingMessageConverter converter = new ContentTypeDelegatingMessageConverter();
   converter.addDelegate(MessageProperties.CONTENT_TYPE_JSON, jsonMessageConverter());
   converter.addDelegate(MessageProperties.CONTENT_TYPE_JSON_ALT, jsonMessageConverter());

   return converter;
}

@Bean
public MessageConverter jsonMessageConverter() {
   return new Jackson2JsonMessageConverter(JsonConverters.nonstrict().getObjectMapper());
}

@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
   SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory() {
      @Override
      protected SimpleMessageListenerContainer createContainerInstance() {
         SimpleMessageListenerContainer container = super.createContainerInstance();
         container.setShutdownTimeout(coreEnvironment.getProperty("amqp.shutdown.timeout", Long::parseLong));
         return container;
      }
   };

   factory.setConnectionFactory(defaultConnectionFactory());
   factory.setAdviceChain(retryInterceptor());
   factory.setMessageConverter(contentTypeDelegatingMessageConverter());
   factory.setConcurrentConsumers(coreEnvironment.getProperty("amqp.consumer.concurrency", Integer::parseInt));
   factory.setMaxConcurrentConsumers(coreEnvironment.getProperty("amqp.consumer.concurrency.max", Integer::parseInt));
   factory.setPrefetchCount(coreEnvironment.getProperty("amqp.consumer.prefetch.count", Integer::parseInt));
   return factory;
}
```

listener에 ContentTypeDelegatingMessageConverter를 적용하면 컨텐츠 타입에 따라 알아서 메시지를 변환해 주지만,  
template에 ContentTypeDelegatingMessageConverter를 적용한다고 해도 원하는 컨텐츠 타입으로 메시지를 변환하기가 어렵다. -> 디폴트 컨버터인 SimpleMessageConverter가 동작해 직렬화/역직렬화만 가능.  
원하는 컨텐츠 타입으로 메시지를 변환해 주려면 어디에선가 ContentType을 셋팅할수 있도록 메소드가 지원되어야 하는데, rabbitTemplate에는 그런 메소드가 없다.  
Object를 변환할 때 무조건 MessageProperties를 새로 만들면서, default content type인 “application/octet-stream” 이 셋팅된다.  
원하는 contentType으로 변환하여 큐에 넣고 싶다면.. rabbitTemplate을 상속받은 클래스를 만들어 사용할 수도 있다.  
그런데.. 좀 무식한 방법인듯...  

```java
public class CustomRabbitTemplate extends RabbitTemplate {
   public void convertAndSend(String routingKey, final Object object, String contentType) throws AmqpException {
      convertAndSend(getExchange(), routingKey, object, (CorrelationData) null, contentType);
   }

   public void convertAndSend(Object object, String contentType) throws AmqpException {
      convertAndSend(getExchange(), getRoutingKey(), object, (CorrelationData) null, contentType);
   }

   public void convertAndSend(String exchange, String routingKey, final Object object, CorrelationData correlationData, String contentType) throws AmqpException {
      send(exchange, routingKey, convertMessageIfNecessary(object, contentType), correlationData);
   }

   private Message convertMessageIfNecessary(final Object object, final String contentType) {
      if (object instanceof Message) {
         return (Message) object;
      }

      MessageProperties properties = new MessageProperties();
      properties.setContentType(contentType);

      return getMessageConverter().toMessage(object, properties);
   }

   public CustomRabbitTemplate(ConnectionFactory connectionFactory) {
      super(connectionFactory);
   }
}


@Bean
public CustomRabbitTemplate rabbitTemplate() {
   CustomRabbitTemplate template = new CustomRabbitTemplate(defaultConnectionFactory());
   template.setMessageConverter(contentTypeDelegatingMessageConverter());
   template.setExchange(coreEnvironment.getProperty("amqp.exchange.direct"));
   return template;
}

```

그냥.. template에는 하나의 기본 MessageConverter만 설정하고, 큐에서 메시지를 받아 쓰는 listener에는 컨텐츠 타입에 따라 유연하게 컨버팅할수 있는 ContentTypeDelegatingMessageConverter를 사용하는 것이 가장 좋을듯….
