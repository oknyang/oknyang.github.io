---
title: 티켓 사용처리 실패 모니터링을 위한 기반데이터 쌓기
date: 2019-10-11 15:16
categories: spring aop
---

티켓을 구매한 사용자가 외부에서 그것을 사용할 경우, 상태를 맞춰주기 위해 우리 시스템의 특정 api를 콜하게 됨.  
우리 api가 처리 도중 실패한 경우를 모니터링하는 배치를 만들고자 함.

아래 두 개의 method가 aspect의 타겟이 됨.  

```java
public class ExternalTicketService {
	public void use(String vendorId, String vendorTicketNo) {
		Validator.validNotEmpty(vendorTicketNo, "연동사티켓번호가 없습니다.");
		TicketMapping mapping = ticketMapper.getAvailableOnly(vendorId, vendorTicketNo);
		Validator.validNotNull(mapping, "티켓매핑정보가 없습니다.");

		Ticketing ticketing = ticketingService.get(mapping);
		Validator.validNotNull(ticketing, "티켓팅정보(%d)가 없습니다.", mapping.getTmonTicketNo());
		Validator.validTrue(TicketingStatus.isUseStatuses(ticketing.getStatus()), "티켓(%d)이 사용처리할 수 있는 상태가 아닙니다: %s", mapping.getTmonTicketNo(), ticketing.getStatus());

		ticketUseService.use(mapping);
	}

	public Map<String, Result<Void>> use(String vendorId, List<String> vendorTicketNos) {
		Validator.validNotEmpty(vendorTicketNos, "연동사티켓번호가 없습니다.");

		List<TicketMapping> mappings = ticketMapper.listAvailableOnly(vendorId, vendorTicketNos);
		Map<String, TicketMapping> mappingMap = ValueUtils.toNullSafeMap(mappings, TicketMapping::getVendorTicketNo);
		Map<Long, Ticketing> ticketingMap = ValueUtils.toNullSafeMap(ticketingService.list(mappings), Ticketing::getTmonTicketNo);
		Map<String, Result<Void>> resultMap = new HashMap<>();

		for (String vendorTicketNo : vendorTicketNos) {
			try {
				TicketMapping mapping = mappingMap.get(vendorTicketNo);
				Validator.validNotNull(mapping, "티켓매핑정보가 없습니다.");

				Ticketing ticketing = ticketingMap.get(mapping.getTmonTicketNo());
				Validator.validNotNull(ticketing, "티켓팅정보(%d)가 없습니다.", mapping.getTmonTicketNo());
				Validator.validTrue(TicketingStatus.isUseStatuses(ticketing.getStatus()), "티켓(%d)이 사용처리할 수 있는 상태가 아닙니다: %s", mapping.getTmonTicketNo(), ticketing.getStatus());
			} catch (Exception e) {
				LOGGER.error(e.getMessage(), e);
				resultMap.put(vendorTicketNo, Result.failure(e));
				mappingMap.remove(vendorTicketNo);
			}
		}

		resultMap.putAll(ticketUseService.use(mappingMap.values()));

		return resultMap;
	}
}
```

아래와 같이 aspect를 만듬.  
use(String, String) method는 성공시 리턴값이 따로 없고,  
실패하면 무조건 exception을 던지도록 되어있기 때문에 `@AfterThrowing` 어노테이션을 이용해 익셉션이 났으면 실패로 간주하고 데이터를 쌓도록 구성함.


use(String, List) method는 복수처리가 가능하기에 일부 성공, 일부 실패의 상황이 있을 수 있고,  
처리중 exception이 발생하는 상황도 있을 수 있기에 둘 다 어드바이스 함.

```java
@Aspect
@Component
public class TicketMonitorAspect {
	@Autowired
	private TicketUseAccumulator accumulator;

	@Pointcut("execution(void com.interwork.api.ticket.service.ExternalTicketService.use(String, String))")
	public void singleTicketUse(){
	}

	@Pointcut("execution(java.util.Map com.interwork.api.ticket.service.ExternalTicketService.use(String, java.util.List))")
	public void ticketUse(){
	}

	@AfterReturning(value = "ticketUse()", returning = "returnValue")
	public void accumulateFail(JoinPoint jp, Map<String, Result<Void>> returnValue) {
		Object[] args = jp.getArgs();
		String vendorId = args[0].toString();
		accumulator.accumulate(vendorId, returnValue);
	}

	@AfterThrowing(value = "singleTicketUse()", throwing = "exception")
	public void accumulateSingleTicketExc0eption(JoinPoint jp, Throwable exception) {
		Object[] args = jp.getArgs();
		String vendorId = args[0].toString();
		List<String> vendorTicketNos = Collections.singletonList(args[1].toString());

		accumulator.accumulate(vendorId, vendorTicketNos, exception);
	}

	@AfterThrowing(value = "ticketUse()", throwing = "exception")
	public void accumulateException(JoinPoint jp, Throwable exception) {
		Object[] args = jp.getArgs();
		String vendorId = args[0].toString();
		List<String> vendorTicketNos = (List<String>) args[1];

		accumulator.accumulate(vendorId, vendorTicketNos, exception);
	}
}
```


