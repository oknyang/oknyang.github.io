---
title: mysql update 시 lock
date: 2020-01-20
categories: spring-data-couchbase couchbase
---

## 정상 처리 프로세스
1. 사용자가 티켓을 구매함
2. 주문에서 게이트웨이에 구매한 티켓에 대해 핀 발급을 요청함.
3. 게이트웨이는 외부사에 핀 발급을 받은 뒤 이를 주문에 콜백으로 돌려줌.
4. 주문에서는 콜백으로 받은 핀을 db에 업데이트함.

## 문제상황
3번 과정에서 주문에 콜백을 주었으나 계속 타임아웃으로 실패 발생함.  
db 업데이트 과정에서 "Lock wait timeout exceeded; try restarting transaction" 오류 발생.

## 원인
데이터 업데이트시 잘못된 인덱스를 탐으로 인해 update lock이 과도하게 잡힘.  

```
InnoDB에서 UPDATE, DELETE 문장을 실행할 때 SQL 문장이 '조건에 일치하는 레코드를 찾기 위해 참조(스캔)하는 인덱스의 모든 레코드에 잠금을 건다'.
여기서 참조(스캔)한 레코드에 잠금을 걸었다는 사실은 실제로 해당 쿼리 문장의 WHERE 조건에 일치하지 않는 레코드도 잠금 대상이 될 수 있음을 의미한다.
InnoDB는 쿼리의 WHERE 절에 명시된 모든 조건에 일치하는 레코드만 선별적으로 잠그는 것이 불가능하다.
```

참고 : https://idea-sketch.tistory.com/47
