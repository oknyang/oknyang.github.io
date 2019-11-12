---
title: "spring-data-couchbase 1.3.0 -> 2.1.6"
date: 2017-01-24
categories: spring-data-couchbase couchbase
---

#### 변경사항 정리

##### Insert 시 익셉션 처리 변경됨.
- 1.3.0 : WriteResultChecking.NONE 이면 익셉션 안던지고 그대로 return
- 2.1.6 : DocumentAlreadyExistsException , CASMismatchException 의 경우 WriteResultChecking.NONE 이어도 익셉션 던짐. (동일 id로 insert시 document가 있으면 DocumentAlreadyExistsException 발생됨)

##### Save 시 익셉션 처리 변경됨.
- 1.3.0 : cas 체크 제대로 안됨. (cas id 없이 새로운 document로 덮어써도 document id만 같으면 덮어써짐.)
- 2.1.6 : cas 체크. cas 없는데 이미 있는 id인 document면 중복 데이터 insert 익셉션 처리처럼 동작함. -> DocumentAlreadyExistsException 익셉션 발생. (cas가 없으니 새로운 document라고 판단하고  insert 하다가 동일 id 존재하니까 insert시 익셉션나는것과 동일하게 처리됨.)
cas가 기존 document와 일치하지 않으면 replace 하다가 CASMismatchException 익셉션 발생

##### Update 로직 변경
- 1.3.0 : 기존 문서 없이 바로 update 가능. (upsert 와 동일하게 동작)
- 2.1.6 : 기존 문서 없이 update 불가. cas 체크 안함. id만 같으면 업데이트 됨.  
