---
title: spring batch xml config -> java config 삽질
date: 2017-02-02
categories: spring-batch 삽질
---

spring batch xml job 설정을 통째로 javaConfig로 바꿔봄.
일단 @EnableBatchProcessing 어노테이션 추가,  
기본적인 컴포넌트 스캔, 잡 런쳐 설정이 이 들어있는.. launch-context.xml 을 @ImportResource 어노테이션으로 읽어오도록 추가 후,  
Spring batch reference 참고하여 job , step  설정함. 기존 xml에 설정된 배치잡 메타정보는 지움.
@EnableBatchProcessing은 일괄 처리 작업을 빌드하기위한 기본 구성을 제공.  
이 기본 구성 내에서 자동으로 사용할 수있게 만들어진 많은 Bean 외에 StepScope의 인스턴스가 만들어진다고 한다.

```java
@Configuration
@EnableBatchProcessing
@ImportResource("classpath:/launch-context.xml")
public class DealMessageDeleteJobConfig {
   @Autowired
   private JobBuilderFactory jobs;

   @Autowired
   private StepBuilderFactory steps;

   @Autowired
   private SqlSessionFactory billingSqlSessionFactory;

   @Autowired
   private SqlSessionFactory batchBillingSqlSessionFactory;

   @Bean
   public Job job(@Qualifier("dealMessageDeleteStep") Step dealMessageDeleteStep) {
      return jobs.get("dealMessageDeleteJob").start(dealMessageDeleteStep).build();
   }

   @Bean
   protected Step dealMessageDeleteStep(ItemReader<String> dealMessageReader, ItemWriter<String> dealMessageWriter) {
      return steps.get("dealMessageDeleteStep")
            .<String, String> chunk(1000)
            .reader(dealMessageReader)
            .writer(dealMessageWriter)
            .build();
   }

   @Bean
   @StepScope
   public ItemReader<String> dealMessageReader(@Value("#{jobParameters['fromDt']}") String fromDt, @Value("#{jobParameters['toDt']}") String toDt) {
      Map<String, Object> paramMap = new HashMap<>();
      paramMap.put("fromDt", fromDt);
      paramMap.put("toDt", toDt);

      MyBatisPagingItemReader<String> reader = new MyBatisPagingItemReader<>();
      reader.setQueryId("selectTicketSrls");
      reader.setSqlSessionFactory(batchBillingSqlSessionFactory);
      reader.setPageSize(10000);
      reader.setParameterValues(paramMap);

      return reader;
   }

   @Bean
   @StepScope
   public ItemWriter<String> dealMessageWriter() {
      MyBatisBatchItemWriter<String> writer = new MyBatisBatchItemWriter<>();
      writer.setSqlSessionFactory(billingSqlSessionFactory);
      writer.setStatementId("updateDealMessage");

      return writer;
   }

}
```

launch-context.xml
```xml
   <context:property-placeholder location="classpath:applicationProperty.properties" />
   <context:component-scan base-package="com.oknyang.batch" />
   <context:component-scan base-package="com.oknyang.core.mail" />
   <context:component-scan base-package="com.oknyang.core.excel" />
   <context:component-scan base-package="com.oknyang.core.api" />
   <context:component-scan base-package="com.oknyang.core.rpc" />
<!--   <bean class="org.springframework.batch.core.scope.StepScope" /> -->
   <import resource="classpath:/springbatch/applicationContext-*.xml" />
   <import resource="classpath:/spring/applicationContext-url.xml"/>

   <bean id="jobLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
      <property name="jobRepository" ref="jobRepository"></property>
   </bean>
 ```

요렇게 수정하고 돌렸더니!  
```
java.lang.IllegalStateException: To use the default BatchConfigurer the context must contain no more thanone DataSource, found 8
```
요런 에러가 남.  
jobRepository를 설정하려면 dataSource가 필요한데, 하나만 있어야 할 데이터 소스가 8개가 발견됐다! 라는 오류..  
정확히 원인은 모르겠으나.. batch_tmon_order 에서 설정한 데이터 소스가 7개이고, tmon-core-batch의 CoreConfiguration 에서 생성하는 데이터 소스가 1개.  
이걸 전부 다 읽어올려서 8개의 데이터 소스가 발견됐다! 라고 에러가 나는것 같은데..  
아직까지 이해가 안되는건.. tmon_batch_order 에서 사용하는 7개의 dataSource 는 각기 다른 id를 줬고,  
core 에서 생성하는 dataSource 는 말 그대로 dataSource 로 생성되는데 이걸 왜 구분하지 못하는지…??  
인젝션 받을때 type 으로 받나..? => SimpleBatchConfiguration에서 초기화시에 BatchConfigurer를 가져오는데, 이때 AbstractBatchConfiguration에서 어플리케이션컨텍스트에 등록된 BatchConfigurer 가 없으면 DefaultBatchConfigurer 를 생성한뒤 데이터 소스가 하나이면 데이터 소스까지 주입해서 넘겨준다.  
그럼 AbstractBatchConfiguration 에서 dataSource를 주입받을때 통으로 가져나보다 싶어서 확인해보니 

```java
@Autowired(required = false)
private Collection<DataSource> dataSources;
```

요렇게 받고 있었음.. DataSource 타입은 몽땅 주입받는.. 이렇게 주입받아놓고 DefaultBatchConfigurer 생성시 데이터 소스가 하나 이상이면 익셉션 발생되게...
참고 - required - @Autowired어노테이션을 적용한 프로퍼티에 대해 굳이 설정할 필요가 없는 경우에 false값을 주며 이때 해당 프로퍼티가 존재하지 않더라도 스프링은 예외를 발생시키지 않는다. 디폴트값은 true

이렇게 되면 DefaultBatchConfigurer 를 사용하면 안된다 싶어서 좀 찾아보니 TmBatchConfigurer 있더라.. core에..  
TmBatchConfigurer 대충 살펴보니 dataSource를 생성자로 하나만 주입할수 있게 해놨음. 

```java
private DataSource dataSource;
public TmBatchConfigurer(DataSource dataSource) {
   setDataSource(dataSource);
}
```

그럼 요걸 어플리케이션 컨텍스트에 등록해서 원하는 dataSource를 주입하면 되겠구나!!! 라는 결론에 도달.
아래와 같이 변경후 테스트해보니 동작함.

```java
@Configuration
@EnableBatchProcessing
@ImportResource("classpath:/launch-context.xml")
public class DealMessageDeleteJobConfig {
   @Autowired
   private JobBuilderFactory jobs;

   @Autowired
   private StepBuilderFactory steps;

   @Autowired
   private SqlSessionFactory billingSqlSessionFactory;

   @Autowired
   private SqlSessionFactory batchBillingSqlSessionFactory;

   @Autowired
   private DataSource dataSource;

   @Bean
   public BatchConfigurer batchConfig() {
      return new TmBatchConfigurer(dataSource);
   }

   @Bean
   public Job job(@Qualifier("dealMessageDeleteStep") Step dealMessageDeleteStep) {
      return jobs.get("dealMessageDeleteJob").start(dealMessageDeleteStep).build();
   }

   @Bean
   protected Step dealMessageDeleteStep(ItemReader<String> dealMessageReader, ItemWriter<String> dealMessageWriter) {
      return steps.get("dealMessageDeleteStep")
            .<String, String> chunk(1000)
            .reader(dealMessageReader)
            .writer(dealMessageWriter)
            .build();
   }

   @Bean
   @StepScope
   public ItemReader<String> dealMessageReader(@Value("#{jobParameters['fromDt']}") String fromDt, @Value("#{jobParameters['toDt']}") String toDt) {
      Map<String, Object> paramMap = new HashMap<>();
      paramMap.put("fromDt", fromDt);
      paramMap.put("toDt", toDt);

      MyBatisPagingItemReader<String> reader = new MyBatisPagingItemReader<>();
      reader.setQueryId("selectTicketSrls");
      reader.setSqlSessionFactory(batchBillingSqlSessionFactory);
      reader.setPageSize(1000);
      reader.setParameterValues(paramMap);

      return reader;
   }

   @Bean
   @StepScope
   public ItemWriter<String> dealMessageWriter() {
      MyBatisBatchItemWriter<String> writer = new MyBatisBatchItemWriter<>();
      writer.setSqlSessionFactory(billingSqlSessionFactory);
      writer.setStatementId("updateDealMessage");

      return writer;
   }


}
```

but, jobRepository 설정할때 테이블 prefix 정하는건 모르겠음
