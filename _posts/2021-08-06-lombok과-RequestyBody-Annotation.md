---
title: lombok과 @RequestyBody
date: 2021-08-06
---

### lombok과 @RequestyBody
사내 스켈레톤 프로젝트를 이용하여 신규 프로젝트 셋팅도중 @RequestBody 어노테이션으로 설정한 오브젝트가 전부 null로 셋팅되는 것을 확인함.  
lombok의 @Getter, @Setter 어노테이션을 사용중이었는데, lombok.config 파일에 아래와 같이 셋팅되어 있는 것을 확인함.  

```
lombok.accessors.fluent = true
```

위 옵션은 getter와 setter 생성시 get, set 프리픽스를 제거하는 역할로, 변수 이름 그대로 getter와 setter가 생성된다.  
반면, ObjectMapper가 객체를 바인딩하기 위해서는 getter나 setter중 반드시 하나는 있어야 하고, get과 set을 제외하고 첫번째 문자를 소문자로 변경하여 필드를 찾는다.  
이때문에 ObjectMapper가 객체를 바인딩하려 getXXX 또는 setXXX를 찾았지만, lombok에서 생성한 getter와 setter의 네이밍 규칙은 달랐고, 이에 ObjectMapper는 필드를 찾지 못해 모든 값이 null로 셋팅되는 상황이 발생했다.  

```
lombok.accessors.fluent = false
```

```
@Accessors(fluent = false)
```

config 파일 옵션을 false로 변경하거나 request 모델에 위와 같이 어노테이션을 추가하여 정상동작함을 확인함.
