---
title: rabbitmq shutdown timeout 설정
date: 2017-12-07
categories: rabbitmq
---

SimpleMessageListenerContainer.doShutdown() 메소드 참고.  

shutdown 동작시 리스너 동작이 끝나지 않았을 경우 지정된 시간동안 대기하다가 그 시간 내에도 작업이 끝나지 않았으면 강제종료 여부를 설정할 수 있음.  

아래는 SimpleMessageListenerContainer.doShutdown() 메소드의 일부.  

```java

logger.info("Waiting for workers to finish.");
boolean finished = this.cancellationLock.await(this.shutdownTimeout, TimeUnit.MILLISECONDS);
if (finished) {
   logger.info("Successfully waited for workers to finish.");
}
else {
   logger.info("Workers not finished.");
   if (isForceCloseChannel()) {
      for (BlockingQueueConsumer consumer : canceledConsumers) {
         consumer.stop();
      }
   }
}

```

문제는 RabbitListener 어노테이션을 사용하기 위해서는 RabbitListenerContainerFactory 설정이 필요한데,  
이 컨테이너 팩토리를 이용해서 shutdownTimeOut을 설정할 방법이 없다는 것…. 
RabbitListener 어노테이션도 사용하고 shutdownTimeOut도 조정하고 싶다면… 직접 SimpleRabbitListenerContainer를 상속받아 구현하는 방법이 있을 수 있다..  
아래는 설정하면 ShutdownTimeout 을 10초로 설정한 예이다.  

```java
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
   SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory() {
      @Override
      protected SimpleMessageListenerContainer createContainerInstance() {
         SimpleMessageListenerContainer container = super.createContainerInstance();
         container.setShutdownTimeout(10_000);

         return container;
      }
   };

   factory.setConnectionFactory(defaultConnectionFactory());
   factory.setAdviceChain(retryInterceptor());
   factory.setMessageConverter(jsonMessageConverter());
   return factory;
}
```
