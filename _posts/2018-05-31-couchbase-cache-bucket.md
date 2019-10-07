---
title: couchbase cache 버킷을 이용한 lock
date: 2018-05-31
categories: couchbase
---

** flow **  
couchbase cache 버킷을 이용해 api 호출 전에 lock을 잡고, 호출이 완료되면 lock을 해제한다.  
lock은 cache 버킷의 document를 이용하는걸로.. (document id로 insert / remove)  

** cache 버킷 : **  
도큐먼트 사이즈 최대 1M. 복제 불가, 리밸런싱 불가. 디스크에 저장되지 않고 RAM에 저장됨. LRU.  
cache 버킷은 n1ql을 지원하지 않음. view도 안됐던거같은데….? 당연히 인덱싱도 안됨.  
https://developer.couchbase.com/documentation/server/4.6/architecture/core-data-access-buckets.html 참고  

** chache 버킷 사용 이유 : **  
이 기능은 어디까지나 흔히들 말하는 횡단 관심사와 유사하기 때문에, 이 기능 하나로 인해 실제 api 실행에 크게 영향이 있어서는 안된다.  
이때문에 insert / remove 등의 동작이 빠를 수록 유리하고, 데이터를 반드시 유지시켜야 할 필요성이 적다.

아래 코드의 insert가 실제 lock을 잡는 역할을 하게 되고, remove가 unlock을 하는 역할이 됨.

```java
@Service
public class DuplicateExecutionPreventService {
   @Qualifier("interworkCacheBucketRepository") @Autowired
   private InterworkCouchbaseRepository interworkCacheBucketRepository;

   public ApiLock insert(ApiType apiType, String identification) {
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

   public String remove(ApiLock model) {
      return interworkCacheBucketRepository.remove(model);
   }
}
```
