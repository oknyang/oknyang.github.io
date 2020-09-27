---
title: java concurrency
date: 2020-09-27
---

### semaphore 생성시 permits 값
permits 값은 점유를 허용하는 숫자인데, 양수 뿐 아니라 0일수도, 음수가 될수도 있다.  
semaphore의 permits 값이 0보다 작거나 같을 경우 permits 값이 최소 1이 될 때까지 release 해줘야 이후 사용이 가능하다.  
ex) 0일 경우 최소 1이 되기 위해 1회 release()  
ex) -1일 경우 최소 1이 되기 위해 2회 release()

### start()와 run()
thread의 run() 메소드를 호출하면 main() 메소드의 콜스택만 사용하게 되기에 제대로된 병렬 처리가 불가능하다.  
start() 메소드는 thread를 runnable 상태로 변환하며, jvm에서 thread를 위한 콜스택을 생성하여 제대로된 병렬 처리가 가능해 진다.

### join()
다른 thread의 종료를 기다림.  
ex) main thread가 다른 작업 thread보다 일찍 끝나는 경우 작업 thread의 수행 완료를 기다리기 위해 사용.

### isAlive()
thread가 살아있는지 체크.
