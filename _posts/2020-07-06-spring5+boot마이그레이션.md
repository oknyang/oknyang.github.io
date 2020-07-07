---
title: spring framework 4 -> spring boot + spring framework 5 마이그레이션
date: 2020-07-06
---

## 과정


## 이슈
### 1. test 코드
rollback을 위해 사용하는 TransactionConfiguration 어노테이션 deprecated됨. 스프링5부터 @Rollback을 클래스레벨에 걸도록 대체됨.
```text
@Rollback may now be used to configure class-level default rollback semantics.
Consequently, @TransactionConfiguration is now deprecated and will be removed in a subsequent release.
```
### 2. java.lang.NoSuchMethodError: ch.qos.logback.core.util.ContextUtil.addHostNameAsProperty()
logback 버전 1.1.7 -> 1.2.3  

### 3. applcationContext bean 오버라이드  
#### 오류
```
The bean 'cacheBase', defined in class path resource [spring/applicationContext-cache.xml], could not be registered. A bean with that name has already been defined in class path resource [spring/applicationContext-ssm.xml] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```
#### 원인
spring boot 2.1 이후부터 bean override가 금지됨. 

#### 해결책
spring.main.allow-bean-definition-overriding 속성값 변경.   
```text
spring.main.allow-bean-definition-overriding
default : false
Whether bean definition overriding, by registering a definition with the same name as an existing definition, is allowed.
```
application.yml에 아래 추가.
```yml
spring:
  main:
    allow-bean-definition-overriding: true
```

### 4. spring-boot, spring-rabbit 호환성 오류
#### 오류
```
[2020-07-06 16:00:05] [RMI TCP Connection(2)-127.0.0.1] [ERROR] o.s.b.SpringApplication.reportFailure[826] Application run failed
java.lang.IllegalStateException: Error processing condition on org.springframework.boot.autoconfigure.amqp.RabbitAnnotationDrivenConfiguration.directRabbitListenerContainerFactoryConfigurer
	at org.springframework.boot.autoconfigure.condition.SpringBootCondition.matches(SpringBootCondition.java:60)
	at org.springframework.context.annotation.ConditionEvaluator.shouldSkip(ConditionEvaluator.java:108)
```
#### 원인
spring-boot-autoconfigure 2.2.6에서 RabbitAnnotationDrivenConfiguration 설정시 DirectRabbitListenerContainerFactory를 생성하는데, 이것이 현재 사용중인 버전 1.7.3에 없음.  
DirectRabbitListenerContainerFactory는 2.0부터 생겨남.  
(https://docs.spring.io/spring-amqp/api/org/springframework/amqp/rabbit/config/DirectRabbitListenerContainerFactory.html) 

#### 해결책
spring-rabbit 1.7.3 → 2.2.8 (버전 조정 필요할 수 있음.)
```xml
<!-- RabbitMQ as AMQP-->
<dependency>
   <groupId>org.springframework.amqp</groupId>
   <artifactId>spring-rabbit</artifactId>
   <version>2.2.8.RELEASE</version>
</dependency>
 
<dependency>
   <groupId>org.springframework.amqp</groupId>
   <artifactId>spring-rabbit-test</artifactId>
   <version>2.2.8.RELEASE</version>
   <scope>test</scope>
</dependency>
```

### 5. spring-boot, spring-data-redis 호환성 오류
#### 오류
```
java.lang.IllegalStateException: Error processing condition on org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration.stringRedisTemplate
    at org.springframework.boot.autoconfigure.condition.SpringBootCondition.matches(SpringBootCondition.java:60)
    at org.springframework.context.annotation.ConditionEvaluator.shouldSkip(ConditionEvaluator.java:108)
    at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsForBeanMethod(ConfigurationClassBeanDefinitionReader.java:184)
    at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsForConfigurationClass(ConfigurationClassBeanDefinitionReader.java:144)
    at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitions(ConfigurationClassBeanDefinitionReader.java:120)
.....
Caused by: java.lang.NoClassDefFoundError: org/springframework/data/redis/connection/jedis/JedisClientConfiguration
    at java.lang.Class.getDeclaredMethods0(Native Method)
    at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)
    at java.lang.Class.getDeclaredMethods(Class.java:1975)
    at org.springframework.util.ReflectionUtils.getDeclaredMethods(ReflectionUtils.java:463)
    ... 78 common frames omitted
```
#### 원인
위 rabbit과 마찬가지로 spring-boot-autoconfigure 2.2.6에서 redis 설정시 JedisClientConfiguration 클래스를 찾는데, spring-data-redis 버전이 맞지 않아 못찾음.  
spring-data-redis는 parent-mvc에 1.7.8로 의존성 설정되어 있음. JedisClientConfiguration은 2.0 부터 추가됨. 

#### 해결책
order_api pom 파일에 아래와 같이 spring-data-redis 오버라이드하고, client 버전도 2.6.0에서 3.3.0으로 올림. (버전 조정 필요할 수 있음.)  
```xml
<!-- redis parent-mvc override -->
<dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-redis</artifactId>
   <version>2.3.1.RELEASE</version>
   <exclusions>
      <exclusion>
         <artifactId>log4j</artifactId>
         <groupId>log4j</groupId>
      </exclusion>
      <exclusion>
         <groupId>org.slf4j</groupId>
         <artifactId>slf4j-log4j12</artifactId>
      </exclusion>
      <exclusion>
         <artifactId>common-logging</artifactId>
         <groupId>common-logging</groupId>
      </exclusion>
   </exclusions>
</dependency>
 
 
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
   <version>3.3.0</version>
   <type>jar</type>
   <scope>compile</scope>
</dependency>
```

### 6. jackson
