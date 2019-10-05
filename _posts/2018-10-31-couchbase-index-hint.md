---
title: couchbase index hint 거는법
date: 2018-10-31
---

from bucket 뒤에 use index (index명 using gsi|view) 붙임.  
but, 잘 적용 안됨…. 이것도 5.0부터인가…  

쿼리 예제
```sql
select * from `interwork_snapshot` use index (SnapshotSimple_orderNos using gsi)
where any orderNo in orderNos satisfies orderNo = 123456789 end
and keyType = 'SNAPSHOT_SIMPLE_REQ'
```
