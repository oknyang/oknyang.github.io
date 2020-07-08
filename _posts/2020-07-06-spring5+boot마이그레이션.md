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

#### 해결책 1
pom 파일에 아래와 같이 spring-data-redis 오버라이드. (버전 조정 필요할 수 있음.)  
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
```
#### 해결책 2
spring boot auto configuration 설정에서 redis 제외
```java
@ImportResource({
      "classpath:mvc-config.xml",
      "classpath:spring/applicationContext-config.xml",
      "classpath:spring/applicationContext-cache.xml",
      "classpath:spring/applicationContext-database.xml",
      "classpath:spring/applicationContext-transaction.xml",
      "classpath:spring/applicationContext-bean.xml",
      "classpath:spring/applicationContext-restTemplate.xml",
      "classpath:spring/applicationContext-couchbase.xml"
})
@SpringBootApplication(exclude = {
      RedisAutoConfiguration.class, // redis 자동설정 제외
      RabbitAutoConfiguration.class,
      CouchbaseAutoConfiguration.class,
      CouchbaseDataAutoConfiguration.class,
      FreeMarkerAutoConfiguration.class,
})
@ComponentScan("com.xxxcorp")
public class OrderApiApplication {
   public static void main(String[] args) {
      SpringApplication.run(OrderApiApplication.class, args);
   }
}
```

### 6. jackson
#### 오류
InvalidDefinitionException 클래스를 찾지 못함.
```
[2020-07-07 15:38:23] [RMI TCP Connection(2)-127.0.0.1] [ERROR] o.s.b.SpringApplication.reportFailure[826] Application run failed
java.lang.NoClassDefFoundError: com/fasterxml/jackson/databind/exc/InvalidDefinitionException
	at java.lang.Class.getDeclaredConstructors0(Native Method)
	at java.lang.Class.privateGetDeclaredConstructors(Class.java:2671)
	at java.lang.Class.getDeclaredConstructors(Class.java:2020)

```
#### 해결책
jackson 버전 2.6.7에서 2.10.3으로 변경
```xml
	<properties>
		<jackson2.version>2.10.3</jackson2.version>
	</properties>
```

### 7. spring-data-couchbase
#### 오류
```
[2020-07-07 17:11:15] [RMI TCP Connection(2)-127.0.0.1] [ERROR] o.s.b.SpringApplication.reportFailure[826] Application run failed
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'applicatorController': Unsatisfied dependency expressed through field 'applicatorService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'applicatorService': Unsatisfied dependency expressed through field 'optionOrderCouchService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'optionOrderCouchService': Unsatisfied dependency expressed through field 'orderCouchService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'orderCouchService': Unsatisfied dependency expressed through field 'orderCouchRepository'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'orderCouchRepository': Unsatisfied dependency expressed through field 'orderCouchbaseTemplate'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'orderCouchbaseTemplate': Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.data.couchbase.core.CouchbaseTemplate]: Constructor threw exception; nested exception is java.lang.NoClassDefFoundError: org/springframework/data/mapping/model/MappingException
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:643)
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:130)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:399)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1422)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:594)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:517)
```
#### 원인
spring 버전업 하면 spring-data-keyvalue와 spring-data-commons가 2.2.6으로 업됨. 이에, spring-data-couchbase에서 의존하는 spring-data-commons 버전이 맞지 않아 오류남.  
spring-data-couchbase는 2.2.10 버전 사용중으로, spring-data-commons 1.3.10 버전을 참조함.
이때문에 couchbase 관련 bean 생성시 spring-data-commons 1.3.10에만 있는 클래스를 찾아서 아래와 같은 오류 발생.  

#### 해결
spring-data-couchbase 버전을 업해야 하는데, 현재 사용중인 버전은 2.X.X 이나, 메이저 버전이 4.X.X까지 올라간 것으로 확인됨. 한번에 2단계의 메이저버전을 올리는것은 위험이 커보여, 일단 3.2.8로 업함. (3.2.8은 spring-data-commons 2.2.8 참조)
```xml
<dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-couchbase</artifactId>
   <version>3.2.8.RELEASE</version>
</dependency>
```

### 8. couchbase 버전 업 후 버킷 생성 오류
#### 오류
```
[2020-07-08 13:10:20] [RMI TCP Connection(2)-127.0.0.1] [ERROR] o.s.b.SpringApplication.reportFailure[826] Application run failed
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'methodValidationPostProcessor' defined in class path resource [org/springframework/boot/autoconfigure/validation/ValidationAutoConfiguration.class]: Unsatisfied dependency expressed through method 'methodValidationPostProcessor' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'orderCouchbaseClientOld': Unsatisfied dependency expressed through constructor parameter 3: Ambiguous argument values for parameter of type [java.lang.String] - did you specify the correct bean references as arguments?
    at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:798)
```
#### 원인
spring-data-couchbase 2 버전에서는 버킷 생성시 cluster, bucketName, bucketPassword 값 필요, 3 버전 올라가면서 username 추가됨.

#### 해결
아래 CouchbaseBucketFactoryBean.java v3.2.8 의 일부 코드를 보면, 버킷이름과 유저이름이 동일하게 넣어주면 됨.
```java
@Override
protected Bucket createInstance() throws Exception {
   if (bucketName == null) {
      return cluster.openBucket();
   }
   else if (password == null) {
      return cluster.openBucket(bucketName);
   }
   else if (bucketName.contentEquals(username)) {
      return cluster.openBucket(bucketName, password);
   }
   else {
      cluster.authenticate(username, password);
      return cluster.openBucket(bucketName);
   }
}
```
couchbase 버킷 설정시 username 추가.
```xml
<couchbase:cluster id="couchbaseClusterOld" env-ref="couchbaseEnvironment">
    <couchbase:node>${couchbase.old.nodes}</couchbase:node>
</couchbase:cluster>
 
<couchbase:clusterInfo id="couchbaseClusterInfoOld" login="${couchbase.old.bucketname}" password="${couchbase.old.password}" cluster-ref="couchbaseClusterOld"/>
 
<!-- username 추가 -->
<couchbase:bucket id="orderCouchbaseClientOld" bucketName="${couchbase.old.bucketname}" bucketPassword="${couchbase.old.password}" username="${couchbase.old.bucketname}" cluster-ref="couchbaseClusterOld"/>
<couchbase:template id="orderCouchbaseTemplateOld" bucket-ref="orderCouchbaseClientOld"/>
 
<!-- username 추가 -->
<couchbase:bucket id="orderHisCouchbaseClientOld" bucketName="${couchbase.old.history.bucketname}" bucketPassword="${couchbase.old.history.password}" username="${couchbase.old.history.bucketname}" cluster-ref="couchbaseClusterOld"/>
<couchbase:template id="orderHisCouchbaseTemplateOld" bucket-ref="orderHisCouchbaseClientOld"/>
```
### 9. HttpMessageConverter 오류
#### 오류
```
[/Users/user/IdeaProjects/service_order_api/deploy/WEB-INF/classes/com/xxxxcorp/api/payment/biz/pay/repository/PgTypeRepository.class]: Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.xxxxcorp.api.payment.biz.pay.repository.PgTypeRepository]: Constructor threw exception; nested exception is java.lang.NullPointerException
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:643)
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:130)
```
#### 원인
HttpMessageConverter 설정하는 부분에서 디폴트 캐릭터셋을 사용하는데, spring-web 5.2.5에서 디폴트 값이 null이 됨.
```java
//AbstractJackson2HttpMessageConverter (spring-web-4.3.3)
public abstract class AbstractJackson2HttpMessageConverter extends AbstractGenericHttpMessageConverter<Object> {
 
   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
 
 
   protected ObjectMapper objectMapper;
//이후 생략
}
```

```java
//AbstractJackson2HttpMessageConverter.java (spring-web-5.2.5)
public abstract class AbstractJackson2HttpMessageConverter extends AbstractGenericHttpMessageConverter<Object> {
 
   /**
    * The default charset used by the converter.
    */
   @Nullable
   @Deprecated
   public static final Charset DEFAULT_CHARSET = null;
 
 
   protected ObjectMapper objectMapper;
// 이후 생략
}
```

#### 해결
4.3.3 버전에서 UTF-8을 default 값으로 사용하고 있었으니, 코드에 직접 UTF-8을 넣어줌.
```java
public class TextAddMappingJackson2HttpMessageConverter extends XxxxMappingJackson2HttpMessageConverter {
    public TextAddMappingJackson2HttpMessageConverter() {
        super(new XxxxObjectMapper());
        this.setSupportedMediaTypes(Arrays.asList(new MediaType[]{
                new MediaType("text", "plain", StandardCharsets.UTF_8),
                new MediaType("application", "json", StandardCharsets.UTF_8),
                new MediaType("application", "*+json", StandardCharsets.UTF_8)}));
    }
}
```

### 10. FreeMarkerConfigurer
#### 오류
```
[2020-07-08 14:11:56] [RMI TCP Connection(2)-127.0.0.1] [ERROR] o.s.b.SpringApplication.reportFailure[826] Application run failed
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'freeMarkerConfigurer' defined in class path resource [org/springframework/boot/autoconfigure/freemarker/FreeMarkerServletWebConfiguration.class]: Invocation of init method failed; nested exception is freemarker.core.Configurable$UnknownSettingException: Unknown FreeMarker configuration setting: "recognize_standard_file_extensions"
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1796)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:595)
```

#### 해결
자동설정 제외. 
