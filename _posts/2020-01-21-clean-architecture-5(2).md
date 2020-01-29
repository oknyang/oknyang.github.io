---
title: "clean architecture 5(2)"
date: 2020-01-21
categories: architecture
---

## 21. 소리치는 아키텍처

### 아키텍처의 테마
이바 야콥슨의 저서 'Object Oriented Software Engineering'에서 야콥슨은 소프트웨어 아키텍처는 시스템의 유스케이스를 지원하는 구조라고 지적했다.  
아키텍처는 프레임워크에 대한 것이 아니다. 아키텍처를 프레임워크로부터 제공받아서는 절대 안된다. 프레임워크는 사용하는 도구일 뿐, 아키텍처가 준수해야 할 대상이 아니다. 아키텍처를 프레임워크 중심으로 만들어버리면 유스케이스가 중심이 되는 아키텍처는 절대 나올 수 없다. 

### 아키텍처의 목적
좋은 소프트웨어 아키텍처는 프레임워크, 데이터베이스, 웹 서버, 그리고 여타 개발 환경 문제나 도구에 대해서는 결정을 미룰 수 있도록 만든다. 좋은 아키텍처는 유스케이스에 중점을 두며, 지엽적인 관심사에 대한 결합은 분리시킨다.  

### 하지만 웹은?
웹은 아키텍처일까? 당연히 아니다! 웹은 전달 메커니즘(입출력 장치)이며, 애플리케이션 아키텍처에서도 그와 같이 다뤄야 한다. 시스템 아키텍처는 시스템이 어떻게 전달될지에 대해 가능하다면 아무것도 몰라야 한다.  

### 프레임워크는 도구일 뿐, 삶의 방식은 아니다.
프레임워크는 매우 강력하고 상당히 유용할 수 있다. 프레임워크 제작자는 자신이 만든 프레임워크를 매우 깊이 신뢰하곤 한다. 이들은 프레임워크를 사용하는 방식을 보여주면서, 흔히 모든 것을 아우르는, 어디에나 스며드는, "프레임워크가 모든것을 하게 하자"라는 태도를 취한다. 이는 우리가 취하고 싶은 태도가 아니다.  
어떻게 하면 아키텍처를 유스케이스에 중점을 둔 채 그대로 보존할 수 있을지를 생각하라. 프레임워크가 아키텍처의 중심을 차지하는 일을 막을 수 있는 전략을 개발하라.  

### 테스트하기 쉬운 아키텍처
아키텍처가 유스케이스를 최우선으로 한다면, 그리고 프레임워크와는 적당한 거리를 둔다면, 프레임워크를 전혀 준비하지 않더라도 필요한 유스케이스 전부에 대해 단위 테스트를 할 수 있어야 한다. 엔티티 객체는 반드시 오래된 방식의 간단한 객체(plain old object)여야 하며, 프레임워크나 데이터베이스, 또는 여타 복잡한 것들에 의존해서는 안 된다. 유스케이스 객체가 엔티티 객체를 조작해야 한다. 최종적으로, 프레임워크로 인한 어려움을 겪지 않고도 반드시 이 모두를 있는 그대로 테스트할 수 있어야 한다. 

### 결론
아키텍처는 시스템을 이야기해야 하며, 시스템에 적용한 프레임워크에 대해 이야기해서는 안 된다.  

## 22. 클린 아키텍처
지난 수십 년간 우리는 시스템 아키텍처와 관련된 여러가지 아이디어들을 봐 왔다. 이들 아키텍처는 모두 관심사의 분리라는 목표를 갖는다. 이들을 소프트웨어를 계층으로 분리함으로써 관심사의 분리라는 목표를 달성할 수 있었다. 각 아키텍처는 최소한 업무 규칙을 위한 계층 하나와, 사용자와 시스템 인터페이스를 위한 또 다른 계층 하나를 반드시 포함한다.  
이들 아키텍처는 모두 시스템이 다음과 같은 특징을 지니도록 만든다.

* 프레임워크 독립성. 아키텍처는 프레임워크의 존재 여부에 의존하지 않는다.
* 테스트 용이성. 업무규칙은 UI, 데이터베이스, 웹 서버, 또는 여타 외부 요소가 없이도 테스트할 수 있다.
* UI 독립성. 시스템의 나머지 부분을 변경하지 않고도 UI를 쉽게 변경할 수 있다. 
* 데이터베이스 독립성. 
* 모든 외부 에이전시에 대한 독립성. 실제로 업무 규칙은 외부 세계와의 인터페이스에 대해 전혀 알지 못한다. 
![clean architecture](https://raw.githubusercontent.com/oknyang/oknyang.github.io/master/the-clean-architecture.jpg "clean architecture")

### 의존성 규칙
위 이미지에서 안으로 들어갈수록 고수준의 소프트웨어가 된다. 바깥쪽 원은 메커니즘이고, 안쪽 원은 정책이다.  
이러한 아키텍처가 동작하도록 하는 가장 중요한 규칙은 의존성 규칙이다.  
```text
소스 코드 의존성은 반드시 안쪽으로, 고수준의 정책을 향해야 한다.
```

내부의 원에 속한 요소는 외부의 원에 속한 어떤 것도 알지 못한다. 같은 이유로, 외부의 원에 선언된 데이터 형식도 내부의 원에서 절대로 사용해서는 안 된다. 특히 그 데이터 형식이 외부의 원에 있는 프레임워크가 생성한 것이라면 더더욱 사용해서는 안 된다.  

#### 엔티티
엔티티는 전사적인 핵심 업무 규칙을 캡슐화한다.  
전사적이지 않은 단순한 단일 애플리케이션을 작성하고 있다면 엔티티는 해당 애플리케이션의 업무 객체가 된다. 이 경우 엔티티는 가장 일반적이며 고수준인 규칙을 캡슐화한다. 외부의 무언가가 변경되더라도 엔티티가 변경될 가능성은 극히 낮다.

#### 유스케이스
유스케이스 계층의 소프트웨어는 애플리케이션에 특화된 업무 규칙을 포함한다. 또한 유스케이스 계층의 소프트웨어는 시스템의 모든 유스케이스를 캡슐화하고 구현한다. 유스케이스는 엔티티로 들어오고 나가는 데이터 흐름을 조정하며, 엔티티가 자신의 핵심 업무 규칙을 사용해서 유스케이스의 목적을 달성하도록 이끈다.  
운영관점에서 애플리케이션이 변경된다면 유스케이스가 영향을 받으며, 따라서 이 계층의 소프트웨어에도 영향을 줄 것이다. 유스케이스의 세부사항이 변하면 이 계층의 코드 일부는 분명 영향받을 것이다. 

#### 인터페이스 어댑터
인터페이스 어댑터 계층은 일련의 어댑터들로 구성된다. 어댑터는 데이터를 유스케이스와 엔티티에게 가장 편리한 형식에서 데이터베이스나 웹 같은 외부 에이전시에게 가장 편리한 형식으로 변환한다.  
마찬가지로 이 계층은 데이터를 엔티티와 유스케이스에게 가장 편리한 형식에서 영속성용으로 사용 중인 임의의 프레임워크(즉, 데이터베이스)가 이용하기에 가장 편리한 형식으로 변환한다.  
또한 이 계층에서는 데이터를 외부 서비스와 같은 외부적인 형식에서 유스케이스나 엔티티에서 사용하는 내부적인 형식으로 변환하는 또 다른 어댑터가 필요하다.  

#### 프레임워크와 드라이버
원의 가장 바깥쪽 계층은 일반적으로 프레임워크나 도구들로 구성된다.  
프레임워크와 드라이버 계층은 모든 세부사항이 위치하는 곳이다.  

#### 원은 네 개여야만 하나?
항상 네 개만 사용해야 한다는 규칙은 없다. 하지만 어떤 경우에도 의존성 규칙은 적용된다. 소스 코드 의존성은 항상 안쪽을 향한다. 안쪽으로 이동할수록 추상화와 정책의 수준은 높아진다. 

#### 경계 횡단하기
그림 우측 하단에 원의 경계를 횡단하는 방법을 보여주는 예시가 있다. 제어흐름과 의존성의 방향이 명백히 반대여야 하는 경우, 대체로 의존성 역전 원칙을 사용하여 해결한다.  
아키텍처 경계를 횡단할 때 언제라도 동일한 기법을 사용할 수 있다. 동적 다형성을 이용하여 소스 코드 의존성을 제어 흐름과 반대로 만들 수 있고, 이를 통해 제어흐름이 어느 방향으로 흐르더라도 의존성 규칙을 준수할 수 있다. 

#### 경계를 횡단하는 데이터는 어떤 모습인가
경계를 가로지르는 데이터는 흔히 간단한 데이터 구조로 이루어져 있다. 중요한 점은 격리되어 있는 간단한 데이터 구조가 경계를 가로질러 전달된다는 사실이다. 꾀를 부려 엔티티 객체나 데이터베이스의 행을 전달하는 일은 원치 않는다. 우리는 데이터 구조가 어떤 의존성을 가져 의존성 규칙을 위배하게 되는 일은 바라지 않는다.  
따라서 경계를 가로질러 데이터를 전달할 때, 데이터는 항상 내부의 원에서 사용하기에 가장 편리한 형태를 가져야만 한다. 

### 결론
소프트웨어를 계층으로 분리하고 의존성 규칙을 준수한다면 본질적으로 테스트하기 쉬운 시스템을 만들게 될 거싱며, 그에 따른 이점을 누릴 수 있다. 데이터베이스나 웹 프레임워크와 같은 시스템의 외부 요소가 구식이 되더라도, 이들 요소를 야단스럽지 않게 교체할 수 있다. 

## 23. 프레젠터와 험블 객체
### 험블 객체 패턴
험블 객체 패턴은 디자인 패턴으로, 테스트하기 어려운 행위와 테스트하기 쉬운 행위를 단위 테스트 작성자가 분리하기 쉽게 하는 방법으로 고안되었다. 행위들을 두 개의 모듈 또는 클래스로 나눈다. 가장 기본적인 본질은 남기고, 테스트하기 어려운 행위를 모두 험블 객체로 옮긴다. 나머지 모듈에는 험블 객체에 속하지 않은, 테스트하기 쉬운 행위를 모두 옮긴다. 
예를 들어 GUI의 경우 단위 테스트가 어려운데, 험블객체 패턴을 사용하면 두 부류의 행위를 분리하여 프레젠터와 뷰라는 서로 다른 클래스로 만들 수 있다.  

### 프레젠터와 뷰
뷰는 험블객체이고 테스트하기 어렵다. 이 객체에 포함된 코드는 가능한 한 간단하게 유지한다. 뷰는 데이터를 GUI로 이동시키지만, 데이터를 직접 처리하지는 않는다.  
프레젠터는 테스트하기 쉬운 객체다. 프레젠터의 역할은 애플리케이션으로부터 데이터를 받아 화면에 표현할 수 있는 포맷으로 만드는 것이다.  
화면에 표시되고 애플리케이션에서 어느 정도 제어할 수 있는 요소라면 무조건 뷰 모델 내부에 문자열, 불, 또는 열거형 형태로 표현한다. 뷰는 뷰 모델의 데이터를 화면으로 로드할 뿐이며, 이 외에 뷰가 맡은 역할은 전혀 없다. 따라서 뷰는 보잘것없다(humble).

### 테스트와 아키텍처
테스트 용이성은 좋은 아키텍처가 지녀야 할 속성으로 오랫동안 알려져 왔다. 험블 객체 패턴이 좋은 얘인데, 행위를 테스트하기 쉬운 부분과 테스트하기 어려운 부분으로 분리하면 아키텍처 경계가 정의되기 때문이다. 프레젠터와 뷰 사이의 경계는 이러한 경계중 하나이며, 이 밖에도 수 많은 경계가 존재한다.  

### 데이터베이스 게이트웨이
유스케이스 인터랙터와 데이터베이스 사이에는 데이터베이스 게이트웨이가 위치한다.  
유스케이스 계층은 SQL을 허용하지 않는다. 따라서 유스케이스 계층은 필요한 메서드를 제공하는 게이트웨이 인터페이스를 호출한다. 그리고 인터페이스 구현체는 데이터베이스 계층에 위치한다. 이 구현체는 험블 객체다. 인터랙터는 애플리케이션에 특화된 업무 규칙을 캡슐화하기 때문에 험블 객체가 아니다. 따라서 게이트웨이는 스텁등으로 적당히 교체하여 쉽게 테스트할 수 있다.

### 데이터 매퍼
하이버네이트 같은 ORM은 어느 계층에 속한다고 보는가? 데이터베이스 계층이다. 실제로 ORM은 게이트웨이 인터페이스와 데이터베이스 사이에서 일종의 또다른 험블 객체 경계를 형성한다.

### 서비스 리스너
애플리케이션이 다른 서비스와 반드시 통신해야 한다면, 또는 애플리케이션에서 일련의 서비스를 제공해야 한다면, 여기에서 서비스 경계를 생성하는 험블 객체 패턴을 발견할 수 있다. 애플리케이션은 데이터를 간단한 데이터 구조 형태로 로드한 후, 이 데이터 구조를 경계를 가로질러서 특정 모듈로 전달한다. 그러면 해당 모듈은 데이터를 적절한 포맷으로 만들어서 외부 서비스로 전송한다. 반대로 외부로부터 데이터를 수신하는 서비스의 경우, 서비스 리스너가 서비스 인터페이스로부터 데이터를 수신하고, 데이터를 애플리케이션에서 사용할 수 있게 간단한 데이터 구조로 포맷을 변경한다. 그런 후 이 데이터 구조는 서비스 경계를 가로질러서 내부로 전달된다.

### 결론
각 아키텍처 경계마다 경계 가까이 숨어 있는 험블 객체 패턴을 발견할 수 있을 것이다. 이러한 아키텍처 경계에서 험블 객체 패턴을 사용하면 전체 시스템의 테스트 용이성을 크게 높일 수 있다.

## 24. 부분적 경계
아키텍처 경계를 완벽학 만드는 데는 비용이 많이 든다. 많은 경우에, 뛰어난 아키텍트라면 이러한 경계를 만드는 비용이 너무 크다고 판단하면서도, 한편으로는 나중에 필요할 수도 있으므로 이러한 경계에 필요한 공간을 확보하기 원할 수도 있다.  
이 문제를 검토하면서 어쩌면 필요할지도 모른다고 판단된다면 부분적 경계를 구현해볼 수 있다.

### 마지막 단계를 건너뛰기
부분적 경계를 생성하는 방법 하나는 독립적으로 컴파일하고 배포할 수 있는 컴포넌트를 만들기 위한 작업은 모두 수행한 후, 단일 컴포넌트에 그대로 모아만 두는 것이다.  
아무리 보아도 이처럼 부분적 경계를 만들려면 완벽한 경계를 만들 때 만큼의 코드량과 사전 설계가 필요하다. 하지만 다수의 컴포넌트를 관리하는 작업은 하지 않아도 된다. 추적을 위한 버전 번호도 없으며, 배포 관리 부담도 없다. 이 차이는 가볍지 않다.  
마지막 단계를 건너뛰는 이 접근법이 지닌 위험성도 있다. 시간이 흐르면서, 별도로 분리한 컴포넌트가 재사용될 가능성은 전혀 없어지고, 분리한 컴포넌트 사이의 구분도 약화되고 의존성은 잘못된 방향으로 선을 넘기 시작할 수 있다.

### 일차원 경계
완벽한 형태의 아키텍처 경계는 양방향으로 격리된 상태를 유지해야 하므로 쌍방향 Boundary 인터페이스를 사용한다. 양방향으로 격리된 상태를 유지하려면 초기 설정할 때나 지속적으로 유지할 때도 비용이 많이 든다.  
추후 완벽한 형태의 경계로 확장할 수 있는 공간을 확보하고자 할 때 전략패턴을 사용하여 간단한 구조를 구현할 수 있다. 그러나 이러한 분리는 쌍방향 인터페이스가 없고 개발자와 아키텍트가 근면 성실하고 제대로 훈련되어 있지 않다면, 매우 빠르게 붕괴될 수 있다.

### 퍼사드
일차원경계보다 훨씬 더 단순한 경계는 페서다 패턴이다. 이 경우에는 심지어 의존성 역전까지도 희생한다. 경계는 Facade 클래스로만 간단히 정의된다. Facade 클래스에는 모든 서비스 클래스를 메서드 형태로 정의하고, 서비스 호출이 발생하면 해당 서비스 클래스로 호출을 전달한다. 클라이언트는 이들 서비스 클래스에 직접 접근할 수 없다.  
하지만 Client가 이 모든 서비스 클래스에 대해 추이 종속성을 가지게 된다. 이는 정적언어였다면 서비스 클래스 중 하나에서 소스 코드가 변경되면 Client도 무조건 재컴파일해야 할 것이다. 

### 결론
위와 같은 접근법 각각은 나름의 비용과 장점을 지닌다. 각 접근법은 완벽한 형태의 경계를 담기 위한 공간으로써, 적절하게 사용할 수 있는 상황이 서로 다르다. 또한 각 접근법은 해당 경계가 실제로 구체화되지 않으면 가치가 떨어질 수 있다.  
아키텍처 경계가 언제, 어디에 존재해야 할지, 그리고 그 경계를 완벽하게 구현할지 아니면 부분적으로 구현할지를 결정하는 일 또한 아키텍트의 역할이다.  

## 25. 계층과 경계





