---
title: verify 삽질 기록
date: 2017-09-07
categories: mockito verify 삽질
---

verify 삽질 기록. 

```java

@RunWith(MockitoJUnitRunner.class)
public class BuyCheckServiceTest {
   @Mock
   private RefCodeService refCodeService;

   @Mock
   private BenefitRepository benefitRepository;

   @Mock
   private AccountService accountService;

   @InjectMocks
   private BuyCheckService buyCheckService;

   @Test
   public void testResetAccountCashAmount_cashAmount가_0일때() {
      AccountWithPoint accountWithPoint = new AccountWithPoint();
      OrderInfo orderInfo = new OrderInfo();
      orderInfo.setAccountWithPoint(accountWithPoint);

      buyCheckService.resetAccountCashAmount(orderInfo);

      verify(accountService, never()).decreaseCashAmount(anyObject(), anyObject());
   }
}

```

위 테스트에서 자꾸 아래와 같은 에러메시지와 함께 테스트 실패가 남.  
이게 뭔 멍멍이 소리여… final 도 , private 도, equals 도, hashCode 도 아닌데…. 한참 ㅅㅂㅅㅂ거리면서 구글링함...

```

Disconnected from the target VM, address: '127.0.0.1:49812', transport: 'socket'

Process finished with exit code 255

org.mockito.exceptions.misusing.UnfinishedVerificationException: 
Missing method call for verify(mock) here:
-> at com.tmoncorp.api.order.biz.buy.service.BuyCheckServiceTest.testResetAccountCashAmount_cashAmount가_0일때(BuyCheckServiceTest.java:899)

Example of correct verification:
    verify(mock).doSomething()

Also, this error might show up because you verify either of: final/private/equals()/hashCode() methods.
Those methods *cannot* be stubbed/verified.


    at org.mockito.internal.runners.util.FrameworkUsageValidator.testFinished(FrameworkUsageValidator.java:25)
    at org.junit.runner.notification.SynchronizedRunListener.testFinished(SynchronizedRunListener.java:56)
    at org.junit.runner.notification.RunNotifier$7.notifyListener(RunNotifier.java:190)
    at org.junit.runner.notification.RunNotifier$SafeNotifier.run(RunNotifier.java:72)
    at org.junit.runner.notification.RunNotifier.fireTestFinished(RunNotifier.java:187)
    at org.junit.internal.runners.model.EachTestNotifier.fireTestFinished(EachTestNotifier.java:38)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:331)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.mockito.internal.runners.JUnit45AndHigherRunnerImpl.run(JUnit45AndHigherRunnerImpl.java:37)
    at org.mockito.runners.MockitoJUnitRunner.run(MockitoJUnitRunner.java:62)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:119)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:42)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:234)
    at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:74)

```

겨우겨우 https://stackoverflow.com/questions/15904584/mockito-gives-unfinishedverificationexception-when-it-seems-ok 요런 글을 찾았다.  
무슨 미완성 검증이 있는 경우 어쩌구 저쩌구 하면서 얘기하는데… 영어 모지리라 잘 모르겠고…. 
아래에 답변 달린걸 보니 테스트 끝나고 validateMockitoUsage() 라는걸 콜하란다.. 그럼 에러메시지가 좀 자세히 보일꺼라는...

```java

@After
public void validate() {
   validateMockitoUsage();
}

```

저걸 붙이고 테스트를 돌려봤더니 아래와 같이 에러메시지가 달라짐.. 아규먼트 매쳐 사용이 잘못되었다는!!!!  
결과적으로, accountService.decreaseCashAmount 메소드는 primitive long 타입의 인자를 받는데,  
내가 쓴 아규먼트 매쳐는 anyObject() 였다.. anyLong() 으로 바꾸고 나니 테스트가 통과하더라...

```

Process finished with exit code 255

org.mockito.exceptions.misusing.InvalidUseOfMatchersException: 
Misplaced argument matcher detected here:

-> at com.tmoncorp.api.order.biz.buy.service.BuyCheckServiceTest.testResetAccountCashAmount_cashAmount가_0일때(BuyCheckServiceTest.java:899)

You cannot use argument matchers outside of verification or stubbing.
Examples of correct usage of argument matchers:
    when(mock.get(anyInt())).thenReturn(null);
    doThrow(new RuntimeException()).when(mock).someVoidMethod(anyObject());
    verify(mock).someMethod(contains("foo"))

Also, this error might show up because you use argument matchers with methods that cannot be mocked.
Following methods *cannot* be stubbed/verified: final/private/equals()/hashCode().


    at org.mockito.internal.runners.util.FrameworkUsageValidator.testFinished(FrameworkUsageValidator.java:25)
    at org.junit.runner.notification.SynchronizedRunListener.testFinished(SynchronizedRunListener.java:56)
    at org.junit.runner.notification.RunNotifier$7.notifyListener(RunNotifier.java:190)
    at org.junit.runner.notification.RunNotifier$SafeNotifier.run(RunNotifier.java:72)
    at org.junit.runner.notification.RunNotifier.fireTestFinished(RunNotifier.java:187)
    at org.junit.internal.runners.model.EachTestNotifier.fireTestFinished(EachTestNotifier.java:38)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:331)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.mockito.internal.runners.JUnit45AndHigherRunnerImpl.run(JUnit45AndHigherRunnerImpl.java:37)
    at org.mockito.runners.MockitoJUnitRunner.run(MockitoJUnitRunner.java:62)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:119)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:42)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:234)
    at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:74)

```
