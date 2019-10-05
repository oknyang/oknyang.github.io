---
title: rabbitmq exclusive 속성과 spring-amqp exclusive 적용
date: 2018-12-06
---

rabbitmq에는 exclusive라는 속성이 존재함.  
일명 큐의 독점적 사용이라는..  

현재 프로젝트에는 리퀘스트시 각 처리 스텝마다 스냅샷을 기록하는 로직이 존재하는데, 이 로직은 크게 보면  
1. 각 요청마다 고유의 id를 따서 이 id는 요청이 끝날때까지 가지고 다닌다. (설사 병렬 스레드로 실행된다 해도 ThreadLoacal에 해당 id를 셋해주어 각 스레드에서도 id를 물고다닐 수 있게 한다.)
2. aspect에서는 이 id를 이용해 해당 스텝의 request, response, header, exception 등의 정보를 가진 모델 객체를 만들어 mq로 보낸다.
3. mq 리스너에서는 메시지를 받아서 couchbase에 도큐먼트를 조회해 보고 도큐먼트가 없으면 저장, 도큐먼트가 있으면 조회된 도큐먼트에 새로 저장해야 할 도큐먼트의 내용을 병합해 저장한다. (upsert)

그냥 aop에서 바로 모델을 저장하면 될 것인데.. 이 로직이 굳이 굳이 이렇게 복잡해 진 이유는 카우치베이스 동시접근 문제 때문이다.  
동일 id를 가진 문서를 여러 스레드에서 동시에 카우치베이스에 저장을 시도하게 되면 도큐먼트가 이미 저장되어 있다는 오류라거나.. cas가 일치하지 않는다거나 하는 데이터 일관성 관련 오류가 발생할 수 있다.  
mq를 사용한다고 해도 리스너가 여러개면 똑같은 문제가 발생하기는 하지만, 리스너를 각 웹서버 인스턴스마다 최소로(1) 지정하면 이러한 현상은 최소로 줄일 수 있지 않을까 해서 mq를 사용하게 되었다.  

잡설 : 여기다가 Qos라고 prefetchCount를 20개로 늘려주면 요청마다 리스너가 프리페치카운트만큼 데이터를 미리 가져가게 되어 이러한 문제는 더욱더 적게 발생할수 있을줄 알았으나...  
prefetchCount는 아무래도 큐에 프리페치 카운트 이상의 메시지가 쌓여 있을때나 유효한 것으로 보인다..  
큐에 메시지가 쌓일 새도 없이 한건씩 들어오면 (리스너가 처리하는 속도가 큐에 메시지가 쌓이는 속도보다 빠르면) 리스너가 메시지를 몰아서 가져갈 건덕지가 없으니...  

다시 본론으로...  
서버 인스턴스가 4개가 떠있으므로 하나의 mq에 리스너가 4개 붙은 상황이고, 이는 여전히 카우치베이스 동시접근 문제가 발생할 수 밖에 없다.  
이걸 어떻게 해결할 방법이 없을까 해서 여기저기 뒤적거려 본 바로는, rabbitmq에 exclusive라는 속성이 있고, 이 속성은 말 그대로 하나의 리스너가 큐를 독점적으로 사용하게 해주는...  
(rabbitmq 책에는 exclusive 속성은 mq에 설정해야 하고, 리스너가 사라지면 큐도 자동으로 삭제된다고 되어있으나.. durable한 큐에도 설정할 수 있는것으로 보이긴 한다.. 정확히는 모르겠다...)

spring-amqp를 확인해 보니 queue에 거는 exclusive 속성은 모르겠고, 리스너에 exclusive 속성을 걸 수 있게 되어있다.  
실제로 테스트 해 보니 큐에 리스너가 하나 등록되고 나면 다른 리스너들은 연결을 시도하다가 거절되어 실패되는 warning 로그가 주구장창 반복적으로 올라오게 된다.  

```java
@RabbitListener(containerFactory = "snapshotListenerContainerFactory", exclusive = true, queues = "test")
public void testListener(String body) throws InterruptedException {
   System.out.println("!@#!@ " + body);
   Thread.sleep(1000);
}
```

큐를 점거하지 못한 listener log

```
[2018-12-05 19:42:12] [WARN ] o.s.a.r.l.SimpleMessageListenerContainer$DefaultExclusiveConsumerLogger.log[1774] Exclusive consumer failure: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=403, reply-text=ACCESS_REFUSED - queue 'test' in vhost 'interwork' in exclusive use, class-id=60, method-id=20)
[2018-12-05 19:42:12] [INFO ] o.s.a.r.l.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run[1690] Restarting Consumer@222a1fb5: tags=[{}], channel=Cached Rabbit Channel: AMQChannel(amqp://interwork@10.11.104.40:5672/interwork,1), conn: Proxy@31e39a00 Shared Rabbit Connection: SimpleConnection@518f6c3c [delegate=amqp://interwork@10.11.104.40:5672/interwork, localPort= 59295], acknowledgeMode=AUTO local queue size=0
[2018-12-05 19:42:12] [INFO ] o.s.a.r.c.CachingConnectionFactory$DefaultChannelCloseLogger.log[1274] Channel shutdown: channel error; protocol method: #method<channel.close>(reply-code=403, reply-text=ACCESS_REFUSED - queue 'test' in vhost 'interwork' in exclusive use, class-id=60, method-id=20)

```

이 로그를 없애보려고.. 아예 등록된 리스너를 빼는 방법이나, 실행 중지 시키는 방법을 찾아보았으나, 찾지 못했다... ㅠ  
참고로 로그는 메소드를 오버라이드하거나 로그레벨을 변경하는 등의 방법으로 찍히지 않도록은 할 수 있을 것 같다.  
하지만 백단에서는 로그 없이 계속 오류를 내고 있겠지....  


SimpleMessageListenerContainer.java 에서 ConditionalExceptionLogger 를 만들어 exclusiveConsumerExceptionLogger 를 set해주면 된다.
```java
private ConditionalExceptionLogger exclusiveConsumerExceptionLogger = new DefaultExclusiveConsumerLogger();

public void setExclusiveConsumerExceptionLogger(ConditionalExceptionLogger exclusiveConsumerExceptionLogger) {
   this.exclusiveConsumerExceptionLogger = exclusiveConsumerExceptionLogger;
}
```
