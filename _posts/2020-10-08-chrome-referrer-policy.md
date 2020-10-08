---
title: chrome 85 referrer default 정책 변경
date: 2020-10-08
---

### referrer로 유입경로를 체크하는 과제 내용 검토 중 이슈 발견

referrer를 체크해 /aaa/bbb 가 경로에 포함되어 있으면 구매를 제한하는 프로세스 개발 검토 요청 유입됨.  
safari에서는 정상동작하나, chrome에서는 referrer에 도메인만 찍혀 내려오는 현상 발생.  
찾아보니 chrome 85 버전 이후부터는 referrer 정책이 변경되어 크로스 도메인이거나, http -> https 등의 상황일 경우 full url이 아닌 도메인 정보만 넘겨주도록 바뀜.  

이에 referrer로 유입경로를 판단하는 부분에서 url에 넘어오는 쿼리 파람으로 유입경로를 판단하기로 변경함.  
사실 이 부분은 위변조가 referrer보다 더욱 쉽다는 문제가 있으나, 구매를 원천 봉쇄 하는 수준으로 막을 필요는 없다는 기획의 판단에 따라 감안하고 진행하기로 함.  

쿼리 파람보다는 구매 상품을 담을때 redis를 이용하는데, 이 때에 유입 경로를 구분할 수 있는 값을 redis에 같이 담아주는것이 좀 더 확실한 방법으로 판단되나, 개발 공수 및 타 도메인 부서의 개발 지원 가능 여부 등의 이슈에 따라 쿼리파람으로 진행하도록 결정됨.  

### referrer 정책

#### no-referrer
referrer 정보 제외하고 요청.

#### no-referrer-when-downgrade
프로토콜이 동일 수준 또는 http -> https일 경우에는 경로와 쿼리파람까지 넘겨준다.  
반대로, https -> http로 보안수준이 낮아질 경우에는 origin (도메인) 정보만 넘겨준다.  
safari, mozilla 등 브라우저 기본 정책. (chrome 85 이하 버전에서도 기본 정책이었음)

#### origin
무조건 origin 값만 넘긴다.  

#### origin-when-cross-origin
동일 도메인일 경우 full url을 넘기고, 그 외에는 origin만 넘긴다.  

#### same-origin
동일 도메인일 경우 referrer를 넘기고, 이외에는 referrer를 넘기지 않는다. 

#### strict-origin
동일 수준의 프로토콜 (http -> http or https -> https)일 경우 referrer를 전송하고, 그 외의 경우에는 넘기지 않는다.

#### strict-origin-when-cross-origin
동일 origin일 경우 full url을 referrer로 전송하고, 동일 수준의 프로토콜일 경우 origin만 referrer로 전송한다.  
그 외에 프로토콜 보안 수준이 더 낮아질 경우에는 referrer를 전송하지 않는다. (chrome 85 이후 default 정책)

#### unsafe-url
무조건 full url을 넘긴다.

### chrome default referrre 정책 변경 방법
chrome://flags/#reduced-referrer-granularity -> disable : stric-origin-when-cross-origin에서 no-referrer-when-downgrade로 변경됨.  
