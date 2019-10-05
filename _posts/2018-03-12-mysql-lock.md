---
title: 동시성 문제 해결을 위한 MySQL 잠금 두가지
date: 2018-03-12
categories: mysql
---

#### 1. lock in share mode
select를 한 후에 트랜잭션이 끝날 때까지 해당 row 값이 변경되지 않을 것을 보장한다.
해당 row를 update하거나 delete 하려는 쿼리는 잠김상태가 되어 트랜잭션이 끝날때까지 대기하게 됨. but, select 쿼리는 얼마든지 여러 세션이 동시 수행 가능.
트랜잭션이 끝나는 시점에 lock이 풀림. auto_commit 꺼야함.

ex) select gold from players where id = 1 LOCK IN SHARE MODE;

#### 2. for update
select로 가져온 데이터를 변경하려고 할 때 사용.
for update는 select로 가져온 해당 row에 대해 다른 세션의 select, update, delete 등의 쿼리가 모두 잠김상태가 됨.
즉, for update를 한 세션 외에 다른 세션들은 모두 해당 row에 접근 할 수 없고 모두 대기상태가 됨.
트랜잭션이 끝나는 시점에 lock이 풀림.

BUT !! deadlock, timeout 등의 발생 가능성때문에 사용하면 안됨..
