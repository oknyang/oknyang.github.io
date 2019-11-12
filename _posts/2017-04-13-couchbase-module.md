---
title: Couchbase n1ql 이용한 데이터 select 로직을 module로 제공하기
date: 2017-04-13
categories: couchbase n1ql module
---

spring-data-couchbase 를 사용하지 않고 couchbase sdk만 이용해서 개발함.  
#### 1. 설정

pom.xml 에 couchbase client sdk 추가.  

```xml
<dependency>
  <groupId>com.couchbase.client</groupId>
  <artifactId>java-client</artifactId>
  <version>2.3.6</version>
</dependency>
```

bucket bean 등록.  
이때, 해당 모듈을 다른 api에서도 사용중이면 빈 생성시 프로퍼티가 없어서 서버가 뜨지 않으므로 orderBucket 빈이 실제 필요할 때에 로딩될 수 있도록 lazy 로딩 시켜줌.

```java
public class CouchbaseConfig {
   private static Logger LOGGER = LoggerFactory.getLogger(CouchbaseConfig.class);

   @Bean
   @Lazy
   public Bucket orderBucket(@Value("${couchbase.nodes}") String nodes, @Value("${couchbase.bucketname}") String bucketName, @Value("${couchbase.password}") String bucketPassword) {
      LOGGER.info("couchbase order bucket creation.. nodes : {}, bucket : {}", nodes, bucketName);

      String[] nodeArray = StringUtils.split(nodes, ",");

      if (ArrayUtils.isEmpty(nodeArray)) {
         nodeArray[0] = nodes;
      }

      return CouchbaseCluster.create(nodeArray).openBucket(bucketName, bucketPassword);
   }
}
```

#### 2. repository


orderBucket을 이용해 n1ql 셀렉트하는 repository 생성. 이 repository도 역시 lazy 로딩.  
findByN1QL은 spring-data-couchbase의 코드 참조함.

```java
@Repository
@Lazy
public class OrderCouchbaseRepository {
   private static final Logger LOGGER = LoggerFactory.getLogger(OrderCouchbaseRepository.class);

   @Autowired
   private TmObjectMapper mapper;

   @Autowired
   private Bucket orderBucket;

   public List<Order> findOrdersWithFuelSurchargeStatusType(String statusType, String startDt, String endDt) throws IOException {
      Statement query = select("meta().id AS _id, meta().cas AS _cas, " + i("order") + ".*").from(i(orderBucket.name()))
            .where(x("_type").eq(s("ORDER"))
                  .and(anyIn("fs", x("fuelSurchargeList")).satisfies(x("fs.statusType").eq(s(statusType))))
                  .and(anyIn("fs", x("fuelSurchargeList")).satisfies(x("fs.saveDt").between(DateFunctions.strToMillis(s(startDt))).and(DateFunctions.strToMillis(s(endDt)))))
            );

      LOGGER.info("Executing Query: {}", query);

      return findByN1QL(N1qlQuery.simple(query), Order.class);

   }


   private <T> List<T> findByN1QL(N1qlQuery n1ql, Class<T> entityClass) throws IOException {

         N1qlQueryResult queryResult = orderBucket.query(n1ql);

         if (queryResult.finalSuccess()) {
            List<N1qlQueryRow> allRows = queryResult.allRows();
            List<T> result = new ArrayList<T>(allRows.size());
            for (N1qlQueryRow row : allRows) {
               JsonObject json = row.value();
               T decoded;
               try {
                  decoded = mapper.readValue(json.toString(), entityClass);
               } catch (IOException e) {
                  LOGGER.error("json deserialize error!!", e);
                  throw e;
               }
               result.add(decoded);
            }
            return result;
         } else {
            StringBuilder message = new StringBuilder("Unable to execute query due to the following n1ql errors: ");
            for (JsonObject error : queryResult.errors()) {
               message.append('\n').append(error);
            }
            throw new RuntimeException(message.toString());
         }
   }
}
```

#### 3. Statement 함수 및 기타 알아두면 좋을 것들

```java
      Statement query = select("meta().id AS _id, meta().cas AS _cas, " + i("order") + ".*").from(i(orderBucket.name()))
            .where(x("_type").eq(s("ORDER"))
                  .and(anyIn("fs", x("fuelSurchargeList")).satisfies(x("fs.statusType").eq(s(statusType))))
                  .and(anyIn("fs", x("fuelSurchargeList")).satisfies(x("fs.saveDt").between(DateFunctions.strToMillis(s(startDt))).and(DateFunctions.strToMillis(s(endDt)))))
            );
```

- i() : `로 감싸기
- x() : quoter, escape 제거
- s() : “”로 감싸기
- DateFuntions.strToMillis() : 카우치베이스에서 제공하는 date function. string을 UNIX millisecond로 변경함.

```
meta().id AS _id, meta().cas AS _cas,
```

요 부분은 spring-data-couchbase를 이용하지 않고 그냥  sdk를 이용해 가져오면 document에 자동으로 셋팅되지 않는듯 하여 추가해줌..  
위 query Statement를 n1ql 쿼리로 변환하면 아래의 Executing Query가 된다.
```
[2017-04-12 17:54:57] [main] [INFO ] c.t.a.o.b.c.r.OrderCouchbaseRepository.findOrdersWithFuelSurchargeStatusType[55] Executing Query: SELECT meta().id AS _id, meta().cas AS _cas, `order`.* FROM `order` WHERE _type = "ORDER" AND ANY fs IN fuelSurchargeList SATISFIES fs.statusType = "AV" END AND ANY fs IN fuelSurchargeList SATISFIES fs.saveDt BETWEEN STR_TO_MILLIS("2017-03-20 00:00:00") AND STR_TO_MILLIS("2017-04-06 00:00:00") END
```

그리고 한가지!!  
__select order.* 과 select * 은 다르다!!__

select *
```
[
  {
    "order": {
      "_class": "com.oknyang.api.order.biz.order.model.Order",
      "_type": "ORDER",
      "createDt": 1491198630535,
      "customDetailList": [],
      "fuelSurchargeList": [
        {
          "accountCouponSrl": 2559204947,
          "accountSrl": 1234567,
          "approveDt": 1491198454000,
          "buySrl": 121232786943,
          "cartCouponAmt": 57280,
``` 

select order.*
```
[
  {
    "_class": "com.oknyang.api.order.biz.order.model.Order",
    "_type": “ORDER",
    "createDt": 1491879122709,
    "customDetailList": [],
    "fuelSurchargeList": [
      {
        "accountSrl": 1234567,
        "approveDt": 1491197157000,
        "buySrl": 121232786175,
        "cartCouponAmt": 0,
```

#### 4. service
OrderCouchbaseService : 역시 lazy loading.. 모듈을 사용하는 api에서 이 orderCouchbaseService 빈을 이용하도록 만듬..

```java
@Component
@Lazypublic
class OrderCouchbaseService {
   @Autowired OrderCouchbaseRepository orderCouchbaseRepository;

   public List<Order> findOrdersWithFuelSurchargeStatusType(String statusType, Date startDt, Date endDt) throws IOException {
      SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
      String strStartDt = sdf.format(startDt);
      String strEndDt = sdf.format(endDt);

      return orderCouchbaseRepository.findOrdersWithFuelSurchargeStatusType(statusType, strStartDt, strEndDt);
   }
}
```

#### 5. 남아있는 문제들..


이렇게 해서.. couchbase를 사용하지 않는 다른 api에서도 영향받지 않고 정상 동작하도록 만들긴 하였으나.. 아직 문제가 해결된게 아님..  
모듈을 만들고, couchbase를 사용중인 api에 이 모듈을 추가하여 테스트 해보면 정상동작 하기는 하나.. 아래와 같이 경고가 발생함.  
```
[2017-04-12 17:54:57] [main] [WARN ] c.c.c.c.l.Slf4JLogger.warn[151] More than 1 Couchbase Environments found (3), this can have severe impact on performance and stability. Reuse environments!
```

카우치베이스 Environment가 1개 이상 발견되었고, 이는 퍼포먼스에 심각한 영향을 초래할 수 있다는.. environment를 재사용 하라는..  
(사실 이건 기본 설정부터 잘못된 부분이 있긴 함. 그동안은 environment 를 따로 설정하지 않았으나, 클러스터나 버켓을 여러개 사용할 경우 default environment를 설정하고 각 버켓들이 이 environment를 사용하도록 해야 하는듯..)

이 문제를 해결하기 위해서는 해당 모듈을 사용하는 api들에서 그냥 bucket 인스턴스를 받아서 사용하는것이 가장 깔끔해 보이나, 카우치베이스 sdk나 스프링 데이터 버전에 영향을 받지 않을 수가 없을것 같아 어려움이 있음.  
아니면 environment bean을 받아서 CouchbaseCluster를 생성하는 방법도 있음!  
문제는.. CouchbaseEnvironment가 언제 최초 생겼는지… (sdk 구버전에서도 CouchbaseEnvironment를 사용하는지.. 구버전에서 CouchbaseEnvironment를 사용하지 않았으면 이 방법은 불가능.. 결국 이 모듈을 사용하는 api의 SDK 버전 호환성이 맞아야함.)

#### 6. 참고

DefaultCouchbaseEnvironment : 
CouchbaseEnvironment의 기본 구현. 이 환경은 재사용되어 AsyncCluster 인스턴스 전체에 전달됩니다. stateful 하며 사용자가 전달한 경우 수동으로 종료해야합니다. 스레드가 관리하는 일부 스레드는 비 데몬 스레드입니다. 디폴트 설정은 Builder 또는 시스템 프로퍼티의 설정에 의해 커스터마이즈 할 수 있습니다. 나중에 생성된 envronment가(?) 항상 우선권을 가지며 런타임시 빌더 설정을 재정의하는 데에도 사용될 수 있습니다.

environment를 받아서 cluster 생성하기  
CouchbaseCluster.java
```java
public static CouchbaseCluster create(final CouchbaseEnvironment environment,
    final String... nodes) {
    return create(environment, Arrays.asList(nodes));
}
```
1번 설정에서 봤던 CouchbaseCluster.create(nodeArray) 코드를 위의 create 메소드로 대체하면 될듯.

