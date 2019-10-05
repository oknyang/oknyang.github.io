---
title: couchbase array index 변수명에 대해..
date: 2018-10-31
---

couchbase array index 생성시 보통 array의 각 item들에 특정 이름을 주고 인덱스를 생성하게 된다.  

```
distinct array var.col for var in items end  
```

위의 var 로 array 인덱스를 생성했으면, **select 시점에도 똑같이 var 변수명을 써서 조회를 해야 array index를 탄다..**  
세상세상 이렇게 멍청할수가….

조회 예)
```
SELECT * FROM `b` 
WHERE ANY var IN items SATISFIES var.col = ‘value' END
```

-> var 라는 이름으로 인덱스를 생성했으면 조회시에도 var라는 이름으로 조회해야 함.....
