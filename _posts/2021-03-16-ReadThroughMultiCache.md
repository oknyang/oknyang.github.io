
---
title: ReadThroughMultiCache
date: 2021-03-16
---

### ReadThroughMultiCache
보통 @ReadThroughMultiCache의 경우 List 인자를 받고, List를 리턴함.  
[1, 2, 3] 을 인자로 전달 하고 [a, b, c]를 리턴받아 캐싱한다면  
보통 [1, 2, 3]을 전체 키로 보고 이와 동일한 인자가 들어와야 캐싱된 값을 내려줄 것 같지만, MultiCache는 인자와 결과값을 key:value로 묶어 아래와 같이 캐싱한다.  
  
1:a , 2:b , 3:c
  
  
