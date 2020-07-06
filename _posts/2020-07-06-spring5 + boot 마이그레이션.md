---
title: spring framework 4 -> spring boot + spring framework 5 마이그레이션
date: 2020-07-06
---

## 과정


## 이슈
### 1. test 코드
rollback을 위해 사용하는 TransactionConfiguration 어노테이션 deprecated됨. 스프링5부터 @Rollback을 클래스레벨에 걸도록 대체됨.
```text
@Rollback may now be used to configure class-level default rollback semantics.
Consequently, @TransactionConfiguration is now deprecated and will be removed in a subsequent release.
```
