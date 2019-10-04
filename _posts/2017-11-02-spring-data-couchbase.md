---
title: spring-data-couchbase CustomConverter
date: 2017-11-02
categories: spring-data-couchbase 삽질
---

spring.io에서는 AbstractCouchbaseDataConfiguration 클래스를 상속받아서 아래 메소드를 오버라이드 하라고 가이드가 나와있으나 AbstractCouchbaseDataConfiguration 클래스를 상속받아 설정하는 방법이 아닌 javaConfig 클래스를 하나 두고 모든 빈을 관리하는 방식을 사용중이라 이 방법을 적용할 수 없다.

```java
@Bean(name = BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
public CustomConversions customConversions() {
  return new CustomConversions(Collections.emptyList());
}
```

그래서 코드를 직접 까봄…. 
일단 CouchbaseTemplate을 생성할 때 CouchbaseConverter를 넘겨줘서 생성할 수 있다는걸 확인함.


```java
public CouchbaseTemplate(final ClusterInfo clusterInfo,
                         final Bucket client,
                         final CouchbaseConverter converter,
                         final TranslationService translationService) {
  this.clusterInfo = clusterInfo;
  this.client = client;
  this.converter = converter == null ? getDefaultConverter() : converter;
  this.translationService = translationService == null ? getDefaultTranslationService() : translationService;
  this.mappingContext = this.converter.getMappingContext();

}
```

CouchbaseTemplate 을 생성할때 converter 관련 로직을 까보면 아래 세 개의 메소드가 연관된다.  

```java
/**
 * Creates a {@link MappingCouchbaseConverter} using the configured {@link #couchbaseMappingContext}.
 *
 * @throws Exception on Bean construction failure.
 */
@Bean(name = BeanNames.COUCHBASE_MAPPING_CONVERTER)
public MappingCouchbaseConverter mappingCouchbaseConverter() throws Exception {
  MappingCouchbaseConverter converter = new MappingCouchbaseConverter(couchbaseMappingContext(), typeKey());
  converter.setCustomConversions(customConversions());
  return converter;
}

/**
 * Creates a {@link CouchbaseMappingContext} equipped with entity classes scanned from the mapping base package.
 *
 * @throws Exception on Bean construction failure.
 */
@Bean(name = BeanNames.COUCHBASE_MAPPING_CONTEXT)
public CouchbaseMappingContext couchbaseMappingContext() throws Exception {
  CouchbaseMappingContext mappingContext = new CouchbaseMappingContext();
  mappingContext.setInitialEntitySet(getInitialEntitySet());
  mappingContext.setSimpleTypeHolder(customConversions().getSimpleTypeHolder());
  mappingContext.setFieldNamingStrategy(fieldNamingStrategy());
  return mappingContext;
}

/**
 * Register custom Converters in a {@link CustomConversions} object if required. These
 * {@link CustomConversions} will be registered with the {@link #mappingCouchbaseConverter()} and
 * {@link #couchbaseMappingContext()}. Returns an empty {@link CustomConversions} instance by default.
 *
 * @return must not be {@literal null}.
 */
@Bean(name = BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
public CustomConversions customConversions() {
  return new CustomConversions(Collections.emptyList());
}
```

일단 customConversions 빈을 등록 (실제로는 couchbaseCustomConversions 라는 이름으로 등록되지만 귀찮으니 메소드명으로..) .
등록된 conversions를 couchbaseMappingConverter 빈을 등록할때 customConversions에 set해서 등록하고,  
couchbaseMappingContext bean을 등록할 때 뭔지는 잘 모르겠지만…. customConversions 빈에서 simpleTypeHolder를 가져와 set한다.  
couchbaseMappingConverter 는 CouchbaseConverter를 상속받아 구현한 것으로, CouchbaseTemplate을 생성할 때 converter 인자로 넘길 수 있다.

여기까지 확인하고 
```java
@Bean
public CouchbaseTemplate interworkCouchbaseTemplate() {
   return new CouchbaseTemplate(couchbaseClusterInfo(), interworkBucket());
}
```

위와같이 등록되는 couchbaseTemplate bean에 컨버터 인자를 넘겨서 생성해주면 되겠다! 라고 생각하고 해보려하니, MappingCouchbaseConverter 를 직접 생성하기가 애매하더라는...  
```java
MappingCouchbaseConverter converter = new MappingCouchbaseConverter(couchbaseMappingContext(), typeKey()); // couchbaseMappingContext(), typeKey()
```
요 부분때문에…  

그래서 방법을 찾다가.. 혹시라도 getter가 있을까 확인해보니 있었다!!!! 그래서 아래와 같이 바꿔줌...

```java
@Bean
public CouchbaseTemplate interworkCouchbaseTemplate() {
    CouchbaseTemplate couchbaseTemplate = new CouchbaseTemplate(couchbaseClusterInfo(), interworkBucket());
    MappingCouchbaseConverter converter = (MappingCouchbaseConverter) couchbaseTemplate.getConverter();
    converter.setCustomConversions(customConversions());
    CouchbaseMappingContext mappingContext = (CouchbaseMappingContext) converter.getMappingContext();
    mappingContext.setSimpleTypeHolder(customConversions().getSimpleTypeHolder());
    return couchbaseTemplate;
}
```

이러면 될 줄 알았으나… 자꾸 컨버터를 안타길래 이상하다 싶어 디버깅까지 해봄..  
정확히는 모르겠으나 실제 컨버팅을 하는 부분은 ConversionService라는 놈이 담당하고 있는데,  
이 놈이 converters 라는 멤버변수를 가지고 있고, 이 멤버변수가 실제 컨버팅 로직을 가진 놈인듯 했다.  
디버깅 해보니 여기에 내가 등록한 컨버터가 보이지 않았다. 그렇다면 ConversionService 의 converters에 컨버터를 추가는 로직을 확인해보자 싶어서 확인함..  

그러다가 아래 코드를 발견.. CustomConversions 클래스에 있는 메소드다..  
```java

/**
 * Populates the given {@link GenericConversionService} with the convertes registered.
 *
 * @param conversionService the service to register.
 */
public void registerConvertersIn(final GenericConversionService conversionService) {
  for (Object converter : converters) {
    boolean added = false;

    if (converter instanceof Converter) {
      conversionService.addConverter((Converter<?, ?>) converter);
      added = true;
    }

    if (converter instanceof ConverterFactory) {
      conversionService.addConverterFactory((ConverterFactory<?, ?>) converter);
      added = true;
    }

    if (converter instanceof GenericConverter) {
      conversionService.addConverter((GenericConverter) converter);
      added = true;
    }

    if (!added) {
      throw new IllegalArgumentException("Given set contains element that is neither Converter nor ConverterFactory!");
    }
  }
}
```

이걸 콜하는 곳을 찾아보니..  
AbstractCouchbaseConverter의 afterPropertiesSet에서 콜하고 있다.  
결국 AbstractCouchbaseConverter를 상속한 MappingCouchbaseConverter 의 afterPropertiesSet을 콜하면 될듯 했다.

```java
@Override
public void afterPropertiesSet() {
  conversions.registerConvertersIn(conversionService);
}
```

그래서 아래와 같이 CouchbaseTemplate bean 등록 메소드에 converter.afterPropertiesSet(); 을 추가함.

```java

@Bean
public CouchbaseTemplate interworkCouchbaseTemplate() {
    CouchbaseTemplate couchbaseTemplate = new CouchbaseTemplate(couchbaseClusterInfo(), interworkBucket());
    MappingCouchbaseConverter converter = (MappingCouchbaseConverter) couchbaseTemplate.getConverter();
    converter.setCustomConversions(customConversions());
    CouchbaseMappingContext mappingContext = (CouchbaseMappingContext) converter.getMappingContext();
    mappingContext.setSimpleTypeHolder(customConversions().getSimpleTypeHolder());
     converter.afterPropertiesSet();
     mappingContext.afterPropertiesSet();
    return couchbaseTemplate;
}
```
---
최종본
```java

@Bean
public CouchbaseTemplate interworkCouchbaseTemplate() {
   CouchbaseTemplate couchbaseTemplate = new CouchbaseTemplate(couchbaseClusterInfo(), interworkBucket());
   MappingCouchbaseConverter converter = (MappingCouchbaseConverter) couchbaseTemplate.getConverter();
   converter.setCustomConversions(customConversions());
   CouchbaseMappingContext mappingContext = (CouchbaseMappingContext) converter.getMappingContext();
   mappingContext.setSimpleTypeHolder(customConversions().getSimpleTypeHolder());
   converter.afterPropertiesSet();
   mappingContext.afterPropertiesSet();
   return couchbaseTemplate;
}

   @Bean
   public CustomConversions customConversions() {
       return new CustomConversions(Arrays.asList(CodeEntityToStringConverter.INSTANCE, StringToCodeEntityConverter.INSTANCE));
   }

   @WritingConverter
   public static enum CodeEntityToStringConverter implements Converter<CodeEntity, String> {
       INSTANCE;

       @Override
       public String convert(CodeEntity codeEntity) {
           return codeEntity.getCode();
       }
   }

   @ReadingConverter
   public static enum StringToCodeEntityConverter implements Converter<String, CodeEntity> {
       INSTANCE;

       @Override
       public CodeEntity convert(String code) {
           return null;
       }
   }
```

여기서 문제는 StringToCodeEntityConverter 를 보면 알수 있듯이.. String을 CodeEntity로 변경하는게 불가능하다..  
CodeEntity는 code를 관리하는 여러 enum 들이 공통으로 구현한 인터페이스인데, enum의 코드 스트링값만 가지고 이게 어떤 enum인지를 판별하기가 불가능하다는…. 
결국 쓸데없는 삽질이었다…. 

그래도 CodeEntityToStringConverter 요건 잘 동작하더라….  

-> ReadingConverter도 각각의 Enum을 컨버터로 등록하면 잘 동작함..
