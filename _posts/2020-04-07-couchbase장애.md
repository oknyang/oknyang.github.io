---
title: couchbase 장애
date: 2020-04-07
---

## couchbase 장애 발생
### 노드 failover 실패
1~4번 노드 중 2, 3번 노드 문제 발생.  
infra에서 2, 3번 노드 failover 시도.  
2번 노드는 정상 제거 되었으나 3번 노드 좀비상태로 제거되지 않고 남아있음.  
이에 강제 failover 시도해야 함. 이럴 경우 일부 데이터 유실 발생할 수 있음.  
현재 운여 시스템은 해당 db에 정책정보를 저장하고 조회하는데, 5분 간격 캐시 적용사용중임.  
이에 임시방편으로 캐시 시간을 20분으로 변경 배포.  
3번 노드 제거시 데이터 유실에 대비하기 위해 couchbase XDCR 기능 이용해 유실되면 안되는 데이터 임시로 개발환경에 복제해 놓음.

## couchbase 장애 발생으로 인한 시스템 오류
### couchbase cache 버킷 장애로 인한 시스템 오류
3번노드 문제 발생으로 cache 버킷 오류 발생.  
상황 인지 시점에는 TimeoutException 발생.  
이에 서버 리스타트 하였으나, 이번에는 Request cancelled in-flight 오류 발생.  
cache 버킷 사용하는 기능은 '중복실행방지' 뿐인데, 이 기능은 실제 시스템 기능엔 크게 영향을 끼치지 않는 부분임.  
이에, 쉽게 해결될 상황이 아닌듯 하여 일단 '중복실행방지' 기능의 ignoreException 부분을 모두 true로 바꿔 해당 기능에서 오류가 발생하더라도 계속 작업을 수행하도로 수정 배포함.  

### cache 버킷 장애 대응 계획
반드시 couchbase cache 버킷을 사용해야할 필요는 없음.  
이에 cache 버킷 장애시 memcached 기반으로 실행 가능하도록 memcached 이용한 기능 개발 예정.  
또한, module에 있던 aspect 관련 코드를 코어 로직은 module로, aspect는 external로 옮겨 장애 발생시 좀 더 빠른 배포가 가능하도록 개선 예정.


