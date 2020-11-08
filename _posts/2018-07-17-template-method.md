---
title: API 중복호출 방지 기능 template method 패턴 적용한 최종본
date: 2018-07-17
---

lock을 잡고, 해제하는 로직을 가진 인터페이스

```java
public interface LockingManager<T> {
   T lock(ApiType apiType, String identification);
   String unlock(T lock);
}
```

위의 인터페이스를 구현한 구현체.  
couchbase를 이용한 lock, unlock 구현.

```java
@Component
public class CouchbaseLockingManager implements LockingManager<ApiLock> {
   @Qualifier("interworkCacheBucketRepository")
   @Autowired
   private InterworkCouchbaseRepository interworkCacheBucketRepository;

   public ApiLock lock(ApiType apiType, String identification) {
      String documentId = KeyMaker.getDupExePreventionKey(apiType, identification);

      ApiLock lock = new ApiLock();
      lock.setKeyType(KeyType.CACHE);
      lock.setId(documentId);
      lock.setApi(apiType.getApiKey());
      lock.setIdentification(identification);
      lock.setDateTime(LocalDateTime.now());

      documentId = interworkCacheBucketRepository.insert(lock);

      return interworkCacheBucketRepository.find(documentId, ApiLock.class);
   }

   public String unlock(ApiLock model) {
      return interworkCacheBucketRepository.remove(model);
   }
}

```

template 클래스.  
lockingManage를 주입받아 lockingManager의  lock, unlock 콜.

```java
public class DuplicateExecutionPreventionTemplate<T> {
   private static final Logger LOGGER = LoggerFactory.getLogger(DuplicateExecutionPreventionTemplate.class);

   private LockingManager<T> lockingManager;

   public DuplicateExecutionPreventionTemplate(LockingManager<T> lockingManager) {
      this.lockingManager = lockingManager;
   }

   public<R> R prevent(PreventionExecutor<R> executor, ApiType apiType, String identification) throws Throwable {
      return prevent(executor, apiType, identification, false);
   }

   public<R> R prevent(PreventionExecutor<R> executor, ApiType apiType, String identification, boolean ignoreException) throws Throwable {
      T lock = lock(apiType, identification, ignoreException);

      if (lock != null) {
         LOGGER.info("### DuplicateExecutionPreventionTemplateTmp. api lock!!!");
      }

      try {
         return executor.execute();
      } finally {
         unlock(lock, ignoreException);

         if (lock != null) {
            LOGGER.info("### DuplicateExecutionPreventionTemplateTmp. api unlock!!!");
         }
      }
   }


   private T lock(ApiType apiType, String identification, boolean ignoreException) {
      try {
         // couchbase cache bucket save -> save 실패시 해당 api 이미 실행중이라는 뜻, 튕겨야함.
         return lockingManager.lock(apiType, identification);
      } catch (OptimisticLockingFailureException e) {
         LOGGER.error(Thread.currentThread().getName() + " " + "### 중복실행 불가. api : {}, id : {}", apiType, identification);
         throw DuplicateExecutionException.of("중복실행 불가. api : %s, id : %s", apiType.getApiKey(), identification);
      } catch (Exception e) {
         if (ignoreException) {
            LOGGER.warn("### DuplicateExecutionPreventionTemplateTmp locking fail.", e);
         } else {
            throw e;
         }
      }

      return null;
   }

   private void unlock(T lock, boolean ignoreException) {
      if (lock == null) {
         LOGGER.warn("### DuplicateExecutionPreventionTemplateTmp unlock. lockObj is null.");
         return;
      }

      try {
         lockingManager.unlock(lock);
      } catch (Exception e) {
         if (ignoreException) {
            LOGGER.warn("### DuplicateExecutionPreventionTemplateTmp unlocking fail.", e); // 중복실행 방지 체크시 couchbase에 장애가 난다해도 실제 로직엔 이상 없도록 warn 찍고 익셉션 무시함.
         } else {
            throw e;
         }
      }
   }
}
```

PreventionExecutor라는 functional interface 생성.  
(굳이 생성할 필요 없이 제공되는 인터페이스 사용해도 되지만.. 그냥 생성함…)

```java
@FunctionalInterface
public interface PreventionExecutor<T> {
   T execute() throws Throwable;
}
```

aspect에서 사용

```java
@Aspect
@Component
public class DuplicateExecutionPreventAspect {
   private CustomExpressionEvaluator evaluator = new CustomExpressionEvaluator();

   @Autowired
   private DuplicateExecutionPreventionTemplate<ApiLock> template;

   @Around("@annotation(DupExePrevention)")
   public Object watchPerformance(ProceedingJoinPoint jp) throws Throwable {
      MethodSignature methodSignature = (MethodSignature) jp.getSignature();
      Method method = methodSignature.getMethod();

      DupExePrevention dupExePrevention = method.getAnnotation(DupExePrevention.class);

      String expression = dupExePrevention.expression();
      boolean ignoreException = dupExePrevention.ignoreException();

      String value = evaluator.getValue(jp, expression, String.class);
      ApiType apiType = dupExePrevention.apiType();

      return template.prevent(jp::proceed, apiType, value, ignoreException);
   }

}
```
