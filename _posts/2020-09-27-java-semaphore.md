---
title: java semaphore permits
date: 2020-09-27
---

### java semaphore 생성시 permits 값
permits 값은 점유를 허용하는 숫자인데, 양수 뿐 아니라 0일수도, 음수가 될수도 있다.  
semaphore의 permits 값이 0보다 작거나 같을 경우 permits 값이 최소 1이 될 때까지 release 해줘야 이후 사용이 가능하다.  
ex) 0일 경우 최소 1이 되기 위해 1회 release()  
ex) -1일 경우 최소 1이 되기 위해 2회 release()
