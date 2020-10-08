---
title: "SELECT 1" Communications link failure
date: 2020-10-08
---

### "SELECT 1" Communications link failure
log4jdbc 모듈을 사용할 경우, 검증 쿼리(select 1)가 간헐적으로 아래 에러를 찍는 문제가 있다.

```text
[2020-10-08 09:44:48] [Timer-2] [ERROR] n.s.l.Slf4jSpyLogDelegator.exceptionOccured[116] org.apache.commons.dbcp.DelegatingStatement.executeQuery(DelegatingStatement.java:208)
2920. select 1
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
The last packet successfully received from the server was 839,997 milliseconds ago. The last packet sent successfully to the server was 0 milliseconds ago.
at sun.reflect.GeneratedConstructorAccessor185.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:404)
at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:981)
at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3465)
at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3365)
at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3805)
at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2478)
at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2625)
at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2547)
at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2505)
at com.mysql.jdbc.StatementImpl.executeQuery(StatementImpl.java:1370)
at net.sf.log4jdbc.StatementSpy.executeQuery(StatementSpy.java:664)
at org.apache.commons.dbcp.DelegatingStatement.executeQuery(DelegatingStatement.java:208)
at org.apache.commons.dbcp.PoolableConnectionFactory.validateConnection(PoolableConnectionFactory.java:658)
at org.apache.commons.dbcp.PoolableConnectionFactory.validateObject(PoolableConnectionFactory.java:635)
at org.apache.commons.pool.impl.GenericObjectPool.evict(GenericObjectPool.java:1528)
at org.apache.commons.pool.impl.GenericObjectPool$Evictor.run(GenericObjectPool.java:1700)
at java.util.TimerThread.mainLoop(Timer.java:555)
at java.util.TimerThread.run(Timer.java:505)
Caused by: java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.
at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:2957)
at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3375)
... 15 common frames omitted
```

이 경우 프로세스 상 ERROR는 아니지만 log4jdbc에서는 ERROR 로그 폭탄을 보내 로그를 보기 어렵게 한다.  

db는 커넥션을 만들어놓고, 특정 시간 동안 사용되지 않으면 강제로 process를 제거한다.  
evictor thread는 대기중인 커넥션이 살아있는지 주기적으로 검증한다. 이 검증에서 실패하면 커넥션 풀에서 제거된다.  
(대기중인 커넥션이 db에 의해 강제 종료되고, evictor가 해당 커넥션을 검증과정을 통해 제거할때 위와 같은 오류 발생.)  
log4jdbc는 이게 검증 과정에서 발생하는 오류인지 아닌지 알 수 없기에 쿼리 실패로 로그를 찍는다.  
검증 자체는 정상적인 절차이고 문제가 없다. 다만, 실제 커넥션 사용량에 비해 유휴 커넥션이 너무 많거나, 검증 관련 설정을 적절히 조절하지 못해면 이러한 실패가 잦을 수 있다.  

log4jdbc를 사용하지 않거나, evictor가 1회 검증시 검증하는 커넥션 갯수를 늘리거나(numTestsPerEvictionRun 값으로 조절 가능), evictor가 스케쥴링 되는 텀(timeBetweenEvictionRunsMillis)을 줄이는 등의 방법으로 해결할 수 있다.  
