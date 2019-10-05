---
title: couchbase n1ql update
date: 2018-10-23
---

#### couchbase n1ql update시 정의되지 않은 field 추가하고자 할때
maxOptionCount **is missing**

```sql
update interwork set maxOptionCount = 200 where keyType = "VENDOR_POLICY" and maxOptionCount is missing returning *
```

#### update시 in절 사용

```sql
update interwork set maxOptionCount = 10000 where keyType = "VENDOR_POLICY" and vendorId in ["A", "B", "C”] returning *
```

#### returning
update 결과를 return하는 구절..  
- returning * : update된 document 전체 조회
- returning count(1) : update된 document 갯수 조회
- returning interwork.maxOptionCount : update된 document의 maxOptionCount 조회
