---
title: "spring ServiceLocatorFactoryBean 사용"
date: 2017-12-27
categories: spring
---

1. applicationContext xml 설정
```
<bean id="converterFactoryBean" class="org.springframework.beans.factory.config.ServiceLocatorFactoryBean">
   <property name="serviceLocatorInterface" value="com.tmoncorp.template.converter.ConverterFactory"/>
</bean>
```

serviceLocatorInterface 는 bean을 가져오는 method를 정의한 interface. bean을 주입받을때에도 serviceLocatorInterface에 set한 ConverterFactory로 주입받는다.  
ServiceLocatorFactoryBean 의  ServiceLocatorInvocationHandler.invokeServiceLocatorMethod 를 보면 serviceLocatorInterface 에 정의한 method의 첫번째 argument 값을 beanName으로 지정해 bean을 가져오는 부분을 확인할 수 있다.  

ConverterFactory에서는 vendorId 값으로 Converter type의 bean을 가져옴.
```
public interface ConverterFactory {
   Converter getConverter(String vendorId);
}
```

ConverterFactory에서 return 할 Converter type의 bean 생성 - sample이라는 이름의 bean, test라는 이름의 bean을 생성함.  

```
public interface Converter<T, S> {
   T convertToExternalRequestParam(InternalRequestParam internalRequestParam);

   InternalResponseParam convertToInternalResponseParam(S externalResponseParam);

   S callVendorOrder(T externalRequestParam);
}
```
```
@Component("sample")
public class SampleConverter implements Converter {

   @Override
   public SampleRequestParam convertToExternalRequestParam(InternalRequestParam internalRequestParam) {
      System.out.println("sample1 convertToExternalRequestParam");

      return new SampleRequestParam();
   }

   @Override
   public InternalResponseParam convertToInternalResponseParam(Object externalResponseParam) {
      System.out.println("sample1 convertToInternalResponseParam");

      return new InternalResponseParam();
   }

   @Override
   public SampleResponseParam callVendorOrder(Object externalRequestParam) {
      return new SampleResponseParam();
   }
}
```
```
@Component("test")
public class Sample2Converter implements Converter {

   @Override
   public SampleRequestParam convertToExternalRequestParam(InternalRequestParam internalRequestParam) {
      System.out.println("test convertToExternalRequestParam");

      return new SampleRequestParam();
   }

   @Override
   public InternalResponseParam convertToInternalResponseParam(Object externalResponseParam) {
      System.out.println("test convertToInternalResponseParam");

      return new InternalResponseParam();
   }

   @Override
   public SampleResponseParam callVendorOrder(Object externalRequestParam) {
      return new SampleResponseParam();
   }
}
```
factory bean의 주입과 사용.  
serviceLocatorInterface로 지정한 ConverterFactory를 주입받았으며, bean을 얻기 위해 converterFactory.getConverter를 콜한다.  
여기에서는 vendorId를 argument로 넘겨주게 되고, 결국 vendorId 와 동일한 name을 가진 bean을 return 받게 된다.   converterFactory.getConverter(“test”) -> Sample2Converter return

```
@Component
public class OrderSendService {
   @Autowired
   private ConverterFactory converterFactory;

   @SuppressWarnings("unchecked")
   public <T, S> InternalResponseParam sendOrder(InternalRequestParam internalRequestParam) {

      Converter converter = converterFactory.getConverter(internalRequestParam.getVendorId());

      T externalRequestParam = (T) converter.convertToExternalRequestParam(internalRequestParam);

      S externalResponseParam = (S) converter.callVendorOrder(externalRequestParam);

      return converter.convertToInternalResponseParam(externalResponseParam);
   }
}
```
