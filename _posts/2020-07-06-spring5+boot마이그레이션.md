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
### 2. java.lang.NoSuchMethodError: ch.qos.logback.core.util.ContextUtil.addHostNameAsProperty()
logback 버전 1.1.7 -> 1.2.3  

### 3. applcationContext bean 오버라이드  
#### 오류
```
The bean 'cacheBase', defined in class path resource [spring/applicationContext-cache.xml], could not be registered. A bean with that name has already been defined in class path resource [spring/applicationContext-ssm.xml] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```
#### 원인
spring boot 2.1 이후부터 bean override가 금지됨. 

#### 해결책
spring.main.allow-bean-definition-overriding 속성값 변경.   
```text
spring.main.allow-bean-definition-overriding
default : false
Whether bean definition overriding, by registering a definition with the same name as an existing definition, is allowed.
```
application.yml에 아래 추가.
```yml
spring:
  main:
    allow-bean-definition-overriding: true
```
