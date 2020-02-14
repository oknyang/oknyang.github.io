---
title: "OncePerRequestFilter"
category: spring
date: 2020-02-14
---

## OncePerRequestFilte
request당 한번만 실행.  
한 request당 한번만 실행되어야 하는 상황에서 이 필터를 사용. Spring Security의 일부 필터도 이 필터를 사용함.  
request당 필터가 여러번 수행되는 상황 : 필터체인에 필터를 두번 이상 가질 수 있음. 또는 request dispatcher를 통해 요청이 다른 서블릿으로 전송될 수도 있음.
