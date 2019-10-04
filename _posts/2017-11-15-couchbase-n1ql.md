---
title: couchbase n1ql timestamp 날짜 검색
date: 2017-11-15
categories: couchbase n1ql
---

couchbase n1ql timestamp 날짜 검색. 

```sql
select * from interwork_snapshot where requestDate between STR_TO_MILLIS('2017-11-14') and STR_TO_MILLIS('2017-11-16')
```
