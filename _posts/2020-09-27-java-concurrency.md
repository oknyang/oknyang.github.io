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

### wait(), notify(), notifyAll()
wait()가 호출되면 알림을 받기 전까지 대기상태가 된다.  
notify()는 대기중인 스레드중 하나에 알림을 준다.  
notifyAll()은 대기중인 스레드를 모두 깨운다.  

#### notify() VS notifyAll()
여러 스레드가 대기 상태일때 notify()를 호출하면, 하나의 스레드에만 알림이 간다. 하지만 어떤 스레드에 알림이 갈지 알 수 없다. (JVM에 의해 결정)  
notifyAll() 호출시 모든 대기 스레드에 알림이 가지만, 어차피 잠금을 획득한 하나의 스레드만 실행된다.  
대기중인 모든 스레드가 상호 교환이 가능한 경우(아마 병행 처리(cuncurrency)일 경우를 말하는 듯)에는 notify()를 사용해야 한다. (ex - thread pool)  
상호배타적인 잠금의 경우 notify()를 사용하는것이 좋다. 제대로 구현했다면 notifyAll()을 사용해도 되지만, 어쨌든 아무것도 할 수 없는 스레드를 불필요하게 깨우게 된다.  
전처리가 완료되면 대기중인 모든 스레드가 작업을 진행할 수 있는 경우에는 notifyAll()을 사용한다.  
참고 url : https://www.geeksforgeeks.org/difference-notify-notifyall-java/

### Thread.yield()
다른 스레드에게 실행을 양보.
