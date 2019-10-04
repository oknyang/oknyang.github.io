---
title: spring-batch ItemReader,ItemWriter
date: 2017-02-02
categories: spring-batch
---

spring batch job task 구성방법은 크게 2가지. 
- Task interface 상속받아 구현,  
- reader processor writer 사용.  

특정 테이블의 컬럼을 삭제하는 배치 구현해야함. task로 한번에 구현 가능하지만 reader , writer 사용하도록 개발해봄.  

초기 xml 설정

```xml
<batch:job-repository id="jobRepository" table-prefix="BATCH_ORDER_" />
<import resource="classpath:/launch-context.xml" 
/>

<bean id="dealMessageReader" class="org.mybatis.spring.batch.MyBatisPagingItemReader" scope="step">
    <property name="sqlSessionFactory" ref="billingSqlSessionFactory"/>
    <property name="pageSize" value="1000" />
    <property name="queryId" value="selectTicketSrls" />
    <property name="parameterValues">
        <map>
            <entry key="fromDt" value="#{jobParameters[fromDt]}"/>
            <entry key="toDt" value="#{jobParameters[toDt]}"/>
        </map>
    </property>
</bean>

<bean id="dealMessageWriter" class="org.mybatis.spring.batch.MyBatisBatchItemWriter" scope="step">
    <property name="sqlSessionFactory" ref="billingSqlSessionFactory"/>
    <property name="statementId" value="updateDealMessage"/>
</bean>

<batch:job id="dealMessageDeleteJob" job-repository="jobRepository">
    <batch:step id="dealMessageDeleteStep">
        <batch:tasklet transaction-manager="transactionManager">
            <batch:chunk reader="dealMessageReader" writer="dealMessageWriter" commit-interval="1000"/>
        </batch:tasklet>
    </batch:step>
</batch:job>
```

MyBatisPagintItemReader, MyBatisBatchItemWriter 사용하여 구현.  
1000 로우씩 끊어 처리하도록 pageSize 1000, commit-interval 1000으로 설정.  
여기서, commit-interval 이 1000이라서 한번에 1000개를 update 칠것 같지만.. 아님.. 1000번 루프돌려서 건건이 업데이트함.  
(이거 생각없이 그냥 in절로 1000건 업데이트하도록 쿼리짰다가 개삽질함..). 
아무튼, 이렇게 설정하면 딱히 repository도, service 로직도 필요 없음. 쿼리 2개만 있으면 끝남.  
but , repository 생성하고 싶지 않았으나 MyBatis는 mapper가 없으면 안되어 어쩔수 없이 껍데기라도 생성함.  

요기서, reader와 writer 설정을 javaConfig로 변경하고 싶어짐…. 

```java
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
```

위처럼 변경, xml 설정에서 reader, writer 설정 지움 -> 잘 동작함..  
요기서 다시 주어진 reader , writer 말고 내가 직접 ItemReader , ItemWriter 만들어보고 싶음..  

```java
@Autowired
private DealMessageRepository dealMessageRepository;

@Bean
@StepScope
public ItemReader<String> dealMessageReader(@Value("#{jobParameters['fromDt']}") String fromDt, @Value("#{jobParameters['toDt']}") String toDt) {
   Map<String, Object> paramMap = new HashMap<>();
   paramMap.put("fromDt", fromDt);
   paramMap.put("toDt", toDt);
   paramMap.put("_pagesize", 1000);

   ItemReader<String> itemReader = new ItemReader<String>() {
      @Override
      public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
         return dealMessageRepository.selectTicketSrls(paramMap);
      }
   };

   return itemReader;
}

@Bean
@StepScope
public ItemWriter<String> dealMessageWriter() {
   ItemWriter<String> itemWriter = new ItemWriter<String>() {
      @Override
      public void write(List<? extends String> items) throws Exception {
         dealMessageRepository.updateDealMessage(items);
      }
   };

   return itemWriter;
}
```
초반엔 내맘대로 String을 input , output으로 줌..  
쿼리는 List로 select하면서.. 이상하다 싶어서 다시 List<String> 을 아웃풋, 인풋으로 받도록 수정.  
초반에 요렇게 고치고 나니.. 뭔가 이상함..  
Collection 이 100개에 그 안에 List가 하나하나 들어있는데 List 사이즈가 1000… Collection에 담긴 List 하나하나의 요소 값은 다 동일..  
이게.. 커밋 인터벌을 100으로 주고 페이지 사이즈를 1000으로 줬더니 느낌상 동일 쿼리를 100번 날려서 동일 결과값을 가진 리스트를 100개 가지고오는거같은 느낌적인 느낌…. 
이걸 어쩌지 하다가.. 그럼 커밋 인터벌을 1로주면.. 한번만 도나 싶어서 커밋인터벌을 1로 변경.
그래도 Collection 안에 List가 담기는 구조는 변하지 않아서 Writer가 받은 input을 그대로 쿼리 변수에 바인딩하려니 안됨..  
(쿼리에서 받는 변수는  List<String> 인데 넘기는 파라미터는 Collection<List<String>>).  
참고할 소스 없나 하다가.. 아래와 같이 변경. items.get(0) 만 뽑아서 던짐 -_-ㅋ

```java
@Autowired
private DealMessageRepository dealMessageRepository;

@Bean
@StepScope
public ItemReader<List<String>> dealMessageReader(@Value("#{jobParameters['fromDt']}") String fromDt, @Value("#{jobParameters['toDt']}") String toDt) {
   Map<String, Object> paramMap = new HashMap<>();
   paramMap.put("fromDt", fromDt);
   paramMap.put("toDt", toDt);
   paramMap.put("_pagesize", 1000);

   ItemReader<List<String>> itemReader = new ItemReader<List<String>>() {
      @Override
      public List<String> read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
         return dealMessageRepository.selectTicketSrls(paramMap);
      }
   };

   return itemReader;
}

@Bean
@StepScope
public ItemWriter<List<String>> dealMessageWriter() {
   ItemWriter<List<String>> itemWriter = new ItemWriter<List<String>>() {
      @Override
      public void write(List<? extends List<String>> items) throws Exception {
         if (items == null || items.size() == 0) {
            throw new RuntimeException("invalid input data!");
         }

         dealMessageRepository.updateDealMessage(items.get(0));
      }
   };

   return itemWriter;
}
```

요렇게 고치고 테스트를 돌려보니 무한루프가 돔.. 확인해보니 ItemReader 종료조건은 null을 리턴할때인데, 그걸 안넣어줌..   

```java
@Bean
@StepScope
public ItemReader<List<String>> dealMessageReader(@Value("#{jobParameters['fromDt']}") String fromDt, @Value("#{jobParameters['toDt']}") String toDt) {
   Map<String, Object> paramMap = new HashMap<>();
   paramMap.put("fromDt", fromDt);
   paramMap.put("toDt", toDt);
   paramMap.put("_pagesize", 1000);

   ItemReader<List<String>> itemReader = new ItemReader<List<String>>() {
      @Override
      public List<String> read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
         List<String> result = dealMessageRepository.selectTicketSrls(paramMap);

         if (CollectionUtils.isEmpty(result)) {
            return null;
         }

         return result;
         //return null;
      }
   };

   return itemReader;
}
```

result가 비었으면  null을 리턴하도록 수정. 요렇게 하고 테스트하니 정상동작함..  
그러나 MyBatisPagingItemReader가 더 깔끔하다 싶어 다시 MyBatisPagingItemReader로 돌아감.. ㅠㅠ

