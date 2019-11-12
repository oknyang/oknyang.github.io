---
title: array index - (simple 생성 관련.. couchbase 5.0 이상 지원)
date: 2018-10-31
---

array inde 생성시 보통  
```
distinct array var for var in items end
```
구문을 많이 쓰는데, 카우치베이스 5.0 버전부터 array index 구문을 simple하게 생성할수 있도록 지원한다.  
(현재 사용버전 4.5인데, 5.0부터 지원한다는 simple 구문을 사용해도 인덱스가 생성되긴 함. 생성되긴 하는데, 별짓을 다 해봐도 인덱스가 걸리지를 않음… ㅠ)  


simple array index 예제
```sql
CREATE INDEX `SnapshotSimple_orderNos`  ON `interwork_snapshot` (DISTINCT `orderNos`)
WHERE keyType = 'SNAPSHOT_SIMPLE_REQ'
```
