---
title: "clean architecture 5"
date: 2020-01-14
categories: architecture
---

## 15. 아키텍처란?
소프트웨어 시스템의 아키텍처란 시스템을 구축했던 사람들ㅇ 만들어낸 시스템의 형태다. 그 모양으 시스템을 컴포넌트로 분할하는 방법, 분할된 컴포넌트를 배치하느 방법, 컴포넌트가 서로 의사소통하는 방식에 따라 정해진다.  
그리고 그 형태는 아키텍처 안에 담긴 소프트웨어 시스템이 쉽게 개발, 배포, 운영, 유지보수되도로 만들어진다.  

```text
이러한 일을 용이하게 만들기 위해서는 가능한 한 많은 선택지를, 가능한 한 오래 남겨두는 전략을 따라야 한다.
```

아키텍처의 주된 목적은 시스템의 생명주기를 지원하는 것이다. 좋은 아키텍처는 시스템을 쉽게 이해하고, 쉽게 개발하며, 쉽게 유지보수하고, 또 쉽게 배포하게 해준다. 아키텍처의 궁극적인 목표는 시스템의 수명과 관련된 비용은 최소화하고, 프로그래머의 생산성은 최대화하는 데 있다.

### 개발
팀 구조가 다르다면 아키텍처 관련 결정에서도 차이가 난다. 팀 규모가 작다면 잘 정의된 컴포넌트나 인터페이스가 없더라도 서로 효율적으로 협력하여 모노리틱(통짜) 시스템을 개발할 수 있다. 이러한 팀으 아키텍처 없이 시작하는데, 팀 규모가 작은 데다가 상위 구조로 인한 장애물이 없기를 바라기 때문이다.  
다르 한편으로 다섯 팀이 시스템을 개발하고 있다면 시스템을 신뢰할 수 있고 안정된 인터페이스를 갖춘, 잘 설계된 컴포넌트 단위로 분리하지 않으면 개발이 진척되지 않는다. 다른 요소를 고려하지 않는다면 이 시스템의 아키텍처는 다섯 개의 컴포넌트로 발전될 가능성이 높다.  
이러한 '팀별 단일 컴포넌트' 아키텍처가 시스템을 배포, 운영, 유지보수하는 데 최적이 가능성은 거의 없다. 그럼에도 여러 팀이 순전히 일정에만 쫓겨서 일한다면, 결국 이 아키텍처로 귀착될 것이다.

### 배포
배포 비용이 높을수록 시스템의 유용성은 떨어진다. 따라서 소프트웨어 아키텍처는 시스템을 단 한번에 쉽게 배포할 수 있도록 만드는 데 그 목표를 두어야 한다.  
안타깝지만 초기 개발 단계에서는 배포 전략을 거의 고려하지 않는다. 이로 인해 개발하기는 쉬울지 몰라도 배포하기는 상당히 어려운 아키텍처가 만들어진다.  
예를 들어 개발 초기 단계에 마이크로서비스 아키텍처를 사용하면 시스템을 쉽게 개발할 수 있다고 생각할 수 있지만, 배포할 시기가 되면 위협적일 만큼 늘어난 수많은 마이크로 서비스를 발견하게 될지도 모른다. 마이크로 서비스들을 서로 연결하기 위해 설정하고 작동 순서를 결정하는 과정에서 오작동이 발생할 원천이 스며들 수도 있기 때문이다.  
만약 아키텍트가 배포 문제를 초기에 고려했다면 이와는 다른 결정을 내렸을 것이다. 더 적은 서비스를 사용하고, 서비스 컴포넌트와 프로세스 수준의 컴포넌트를 하이브리드 형태로 융합하며, 좀 더 통합된 도구를 사용하여 상호 연결을 관리했을 것이다.

### 운영
운영에서 겪는 대다수의 어려움은 소프트웨어 아키텍처에는 극적인 영향을 주지 않고도 단순히 하드웨어를 더 투입해서 해결할 수 있다.  
하드웨어는 값싸고 인력은 비싸다는 말이 뜻하는 바는 운영을 방해하는 아키텍처가 개발, 배포, 유지보수를 방해하는 아키텍처보다는 비용이 덜 든다는 뜻이다.  
시스템 운영을 쉽게 해주는 아키텍처가 바람직하지 않다는 말은 아니다. 다만 비용 공식 관점에서 운영보다는 개발, 배포, 유지보수쪽으로 더 기운다는 말이다.  
시스템 아키텍처는 유스케이스, 기능, 시스템의 필수 행위를 일급 엔티티로 격상시키고, 이들 요소가 개발자에게 주요 목표로 인식되도록 해야 한다. 이를 통해 시스템을 이해하기 쉬워지며, 따라서 개발과 유지보수에 큰 도움이 된다.

### 유지보수
유지보수는 모든 측면에서 봤을 때 소프트웨어 시스템에서 비용이 가장 많이 든다.  
유지보수의 가장 큰 비용은 탐사와 이로 인한 위험부담에 있다. 탐사란 기존 소프트웨어에 새로운 기능을 추가하거나 결함을 수정할 때, 소프트웨어를 파헤쳐서 어디를 고치는 게 최선인지, 그리고 어떤 전략을 쓰는게 최적일지를 결정할 때 드는 비용이다.  
주의를 기울여 신중하게 아키텍처를 만들면 이 비용을 크게 줄일 수 있다. 시스템을 컴포넌트로 분리하고, 안정된 인터페이스를 두어 서로 격리한다. 이를 통해 미래에 추가될 가능성에 대한 길을 밝혀 둘 수 있을 뿐 아니라 의도치 않은 장애가 발생할 위험을 크게 줄일 수 있다.

### 선택사항 열어 두기
소프트웨어는 행위적 가치와 구조적 가치를 지닌다. 소프트웨어를 부드럽게 만드는 것은 이들 중 구조적 가치이다.  
소프트웨어를 부드럽게 유지하는 방법은 선택사항을 가능한 한 많이, 그리고 가능한 한 오랫동안 열어 두는 것이다. 그렇다면 열어 둬야 할 선택사항이란 무엇일까? 그것은 바로 중요치 않은 세부사항이다.  
모든 소프트웨어 시스템은 주요한 두 가지 구성요소로 분해할 수 있다. 바로 정책과 세부사항이다. 정책 요소는 모든 업무 규칙과 업무 절차를 구체화 한다. 세부사항은 사람, 외부 시스템, 프로그래머가 정책과 소통할 때 필요한 요소지만, 정책이 가진 행위에는 조금도 영향을 미치지 않는다. (입출력장치, 데이터베이스, 웹 시스템, 서버, 프레임워크, 통신 프로토콜 등)  
아키텍트의 목표는 시스템에서 정책을 가장 핵심적인 요소로 식별하고, 동시에 세부사항은 정책에 무관하게 만들 수 있는 형태의 시스템을 구축하는 데 있다.  
세부사항에 몰두하지 않은 채 고수준의 정책을 만들 수 있다면, 이러한 세부사항에 대한 결정을 오랫동안 미루거나 연기할 수 있다. 선택사항을 더 오랫동안 열어둘 수 있다면 더 많은 실험을 해볼 수 있고 더 많은 것을 시도할 수 있다.  
```text
좋은 아키텍트는 결정되지 않은 사항의 수를 최대화한다.
```

### 결론
좋은 아키텍트는 세부사항을 정책으로부터 신중하게 가려내고, 정책이 세부사항과 결합되지 않도록 엄격하게 분리한다. 이를 통해 정책은 세부사항에 관한 어떠한 지식도 갖지 못하게 되며, 어떤 경우에도 세부사항에 의존하지 않게 된다. 좋은 아키텍트는 세부사항에 대한 결정을 가능한 한 오랫동안 미룰 수 있는 방향으로 정책을 설계한다.

## 16. 독립성
좋은 아키텍처는 다음을 지원해야 한다.  
* 시스템의 유스케이스
* 시스템의 운영
* 시스템의 개발
* 시스템의 배포

### 유스케이스
첫 번째 주요 항목인 유스케이스의 경우, 시스템의 아키텍처는 시스템의 의도를 지원해야 한다는 뜻이다. 아키텍트의 최우선 관심사는 유스케이스이며, 아키텍처에서도 유스케이스가 최우선이다. 아키텍트는 반드시 유스케이스를 지원해야 한다.  
좋은 아키텍처가 행위를 지원하기 위해 할 수 있는 일 중에서 가장 중요한 사항은 행위를 명확히 하고 외부로 드러내며, 이를 통해 시스템이 지닌 의도를 아키텍처 수준에서 알아볼 수 있게 만드는 것이다.

### 운영
시스템의 운영 지원 관점에서 볼 때 아키텍처는 더 실질적이며 덜 피상적인 역할을 맡는다. 시스템이 초당 100,000명의 고객을 처리해야 한다면, 아키텍처는 이 요구와 관련된 각 유스케이스에 걸맞은 처리량과 응답시간을 보장해야 한다. 만약 시스템에서 수 밀리초 안에 3차원의 빅데이터테이블에 질의해야 한다면, 반드시 이러한 운영 작업을 허용할 수 있는 형태로 아키텍처를 구조화해야 한다.  
이러한 형태를 지원한다는 말은 시스템에 따라 아키텍처를 다양하게 구조화 할 수 있다는 의미를 지닌다. 아키텍처 구조화에 대한 결정은 뛰어난 아키텍트라면 열어 두어야 하는 선택사항 중의 하나다. 

### 개발
아키텍처는 개발환경을 지원하는 데 있어 핵심적인 역할을 수행한다.  
```text
콘웨이의 법칙
시스템을 설계하는 조직이라면 어디든지 그 조직의 의사소통 구조와 동일한 구조의 설계를 만들어 낼 것이다.
```
많은 팀으로 구성되며 관심사가 다양한 조직에서 어떤 시스템을 개발해야 한다면, 각 팀이 독립적으로 행동하기 편한 아키텍처를 반드시 확보하여 개발하는 동안 팀들이 서로를 방해하지 않도록 해야 한다. 이러한 아키텍처를 만들려면 잘 격리되어 독립적으로 개발 가능한 컴포넌트 단위로 시스템을 분할할 수 있어야 한다. 그래야만 이들 컴포넌트를 독립적으로 작업할 수 있는 팀에 할당할 수 있다.

### 배포
아키텍처는 배포 용이성을 결정하는데 중요한 역할을 한다. 이때 목표는 '즉각적인 배포'다. 좋은 아키텍처라면 시스템이 빌드된 후 즉각 배포할 수 있도록 지원해야 한다.  
이러한 아키텍처를 만들려면 시스템을 컴포넌트 단위로 적절하게 분할하고 격리시켜야 한다. 마스터 컴포넌트는 시스템 전체를 하나로 묶고, 각 컴포넌트를 올바르게 구동하고 통합하고 관리해야 한다.

### 선택사항 열어놓기
좋은 아키텍처는 선택사항을 열어둠으로써, 향후 시스템에 변경이 필요할 때 어떤 방향으로든 쉽게 변경할 수 있도록 한다.

### 계층 결합 분리
아키텍트는 단일 책임 원칙과 공통 폐쇄 원칙을 적용하여, 그 의도의 맥락에 따라서 다른 이유로 변경되는 것들은 분리하고, 동일한 이유로 변경되는 것들은 묶는다.  
업무 규칙은 그 자체가 애플리케이션과 밀접한 관련이 있거나, 혹은 더 범용적일 수도 있다. 이들 서로 다른 두 유형의 규칙은 각자 다른 속도로, 그리고 다른 이유로 변경될 것이다. 따라서 이들 규칙은 서로 분리하고, 독립적으로 변경할 수 있도록 만들어야만 한다.  

### 유스케이스 결합 분리
유스케이스 그 자체 또한 서로 다른 이유로 변경될 수 있다. 주문 입력 시스템에서 주문을 추가하는 유스케이스는 주문을 삭제하는 유스케이스와는 틀림없이 다른 속도로, 그리고 다른 이유로 변경된다. 유스케이스는 시스템을 분할하는 매우 자연스러운 방법이다.  
이와 동시에 유스케이스는 시스템의 수평적인 계층을 가로지르도록 자른, 수직으로 좁다란 조각이기도 하다. 이와 같이 결합을 분리하려면 주문 추가 유스케이스의 UI와 주문 삭제 유스케이스의 UI를 분리해야 한다. 유스케이스의 업무 규칙과 데이터베이스 부분도 마찬가지다. 이런 식으로 시스템의 맨 아래 계층까지 수직으로 내려가며 유스케이스들이 각 계층에서 서로 겹치지 않게 한다.  
여기에서 패턴을 볼 수 있다. 시스템에서 서로 다른 이유로 변경되는 요소들의 결합을 분리하면 기존 요소에 지장을 주지 않고도 새로운 유스케이스를 계속해서 추가할 수 있게 된다. 

### 결합 분리 모드
이렇게 결합을 분리하면 운영 관점에서 어떤 의미가 있는지 살펴보기로 하자. 유스케이스를 위해 수생하는 결합 분리 작업들은 운영에도 도움이 된다. 하지만 운영 측면에서 이점을 살리기 위해선 결합을 분리할 때 적절한 모드를 선택해야 한다. 예를 들어 분리된 컴포넌트를 서로 다른 서버에서 실행해야 하는 상황이라면, 이들 컴포넌트가 단일 프로세서의 동일한 주소 공간에 함께 상주하는 형태로 만들어져서는 안 된다. 분리된 컴포넌트는 반드시 독립된 서비스가 되어야 하고, 일종의 네트워크를 통해 서로 통신해야 한다.  
많은 아키텍트가 이러한 컴포넌트를 '서비스' 또는 '마이크로 서비스'라고 하는데, 그 구분 기준은 모호한 면이 있다. 실제로 서비스에 기반한 아키텍처를 흔히들 서비스 지향 아키텍처라고 부른다.  
여기서 얘기하고자 하는 핵심은, 우리는 때때로 컴포넌트를 서비스 수준까지도 분리해야 한다는 것이다.  
기억해야 할 점은 좋은 아키텍처는 선택권을 열어 둔다는 사실이다. 결합 분리 모드는 이러한 선택지 중 하나다. 

### 개발 독립성
컴포넌트가 완전히 분리되면 팀 사이의 간섭은 줄어든다. 유스케이스와 계층의 결합이 분리되는 한 시스템의 아키텍처는 그 팀 구조를 뒷받침해 줄 것이다.

### 배포 독립성
유스케이스와 계층의 결합이 분리되면 배포 측면에서도 고도의 유연성이 생긴다. 실제로 결합을 제대로 분리했다면 운영 중인 시스템에서도 계층과 유스케이스를 교체 할 수 있다.

### 중복
소프트웨어에서 중복은 일반적으로 나쁜 것이다. 하지만 중복에도 여러 종류가 있다. 그 중 하나는 진짜 중복이다. 이 경우 한 인스턴스가 변경되면, 동일한 변경을 그 인스턴스의 모든 복사본에 반드시 적용해야 한다. 또 다른 중복은 거싲된 또는 우발적인 중복이다. 중복으로 보이는 두 코드 영역이 각자의 경로로 발전한다면, 즉 서로 다른 속도와 다른 이유로 변경된다면 이 두 코드는 진짜 중복이 아니다.  
두 유스케이스의 화면 구조가 매우 비슷하다고 가정해 보자. 이는 우발적 중복일 가능성이 높다. 시간이 지나면서 두 화면은 서로 다른 방향으로 분기하며, 결국에는 매우 다른 모습을 가질 가능성이 높다. 이러한 이유로 해당 코드를 통합하지 않도록 유의해야 한다. 그렇지 않으면 나중에 코드를 다시 분리하느라 큰 수고를 감수해야 한다.  
유스케이스를 수직으로 분리할 때 이런 문제와 마주칠 테고, 이들 유스케이스를 통합하고 싶다는 유혹을 받게 될 것이다. 왜냐하면 이들 유스케이스가 서로 비슷한 화면 구조, 비슷한 알고리즘, 그리고 비슷한 데이터베이스 쿼리와 스키마를 가지기 때문이다. 조심하라. 자동반사적으로 중복을 제거해버리는 잘못을 저지르는 유혹을 떨쳐내라. 중복이 진짜 중복인지 확인하라.  
마찬가지로 계층을 수평으로 분리하는 경우, 특정 데이터베이스 레코드의 데이터 구조가 특정 화면의 데이터 구조와 상당히 비슷하다는 점을 발견할 수도 있다. 이때 데이터베이스 레코드와 동일한 형태의 뷰 모델을 만들어서 각 항목을 복사하는게 아니라, 데이터베이스 레코드를 있는 그대로 UI까지 전달하고 싶다는 유혹을 받을 수도 있다. 조심하라. 이러한 중복은 거의 확실히 우발적이다. 뷰 모델을 별도로 만드는 일은 그다지 많은 노력이 들지 않을 뿐만 아니라, 계층 간 결합을 적절하게 분리하여 유지하는 데도 도움이 될 것이다.

### 결합 분리 모드(다시)
계층과 유스케이스의 결합을 분리하는 방법은 다양하다.  
* 소스 수준 분리 모드 : 소스 코드 모듈 사이의 의존성을 제어할 수 있다. 이를 통해 하나의 모듈이 변하더라도 다른 모듈을 변경하거나 재컴파일하지 않도록 만들 수 있다. 이 모드에서는 모든 컴포넌트가 같은 주소 공간에서 실행되고, 컴퓨터 메모리에는 하나의 실행 파일만이 로드된다. 이러한 구조를 흔히 모노리틱 구조라고 부른다.
* 배포 수준 분리 모드 : jar 파일, DLL, 공유 라이브러리와 같이 배포 가능한 단위들 사이의 의존성을 제어할 수 있다. 이 모드의 중요한 특징은 결합이 분리된 컴포넌트가 jar 파일, Gem 파일, DLL과 같이 독립적으로 배포할 수 있는 단위로 분할되어 있다는 점이다.
* 서비스 수준 분리 모드 : 의존하는 수준을 데이터 구조 단위까지 낮출 수 있고, 순전히 네트워크 패킷을 통해서만 통신하도록 만들 수 있다. 이를 통해 모든 실행 가능한 단위는 소스와 바이너리 변경에 대해 서로 완전히 독립적이게 된다.  

좋은 아키텍처는 시스템이 모노리틱 구조로 태어나서 단일 파일로 배포되더라도, 이후에는 독립적으로 배포 가능한 단위들의 집합으로 성장하고, 또 독립적인 서비스나 마이크로 서비스 수준까지 성장할 수 있도록 만들어져야 한다. 또한 좋은 아키텍처라면 나중에 상황이 바뀌었을 때 이 진행 방향을 거꾸로 돌려 원래 형태인 모노리틱 구조로 되돌릴 수도 있어야 한다.  
좋은 아키텍처는 이러한 변경으로부터 소스 코드 대부분을 보호한다. 좋은 아키텍처는 결합 분리 모드를 선택사항으로 남겨두어서 배포 규모에 따라 가장 적합한 모드를 선택해 사용할 수 있게 만들어 준다. 

### 결론
시스템의 결합 분리 모드는 시간이 지나면서 바뀌기 쉬우며, 뛰어난 아키텍트라면 이러한 변경을 예측하여 큰 무리 없이 반영할 수 있도록 만들어야 한다. 

## 17. 경계: 선 긋기
소프트웨어 아키텍처는 선을 긋는 기술이며, 이러한 선을 경계라고 부른다. 경계는 소프트웨어 요소를 서로 분리하고, 경계 한편에 있는 요소가 반대편에 있는 요소를 알지 못하도록 막는다.  
아키텍트의 목표는 필요한 시스템을 만들고 유지하는 데 드는 인적 자원을 최소화 하는것이다. 그렇다면 인적 자원의 효율을 떨어뜨리는 요인은 무엇일까? 바로 결합이다. 특히 너무 일찍 내려진 결정에 따른 결합이다.  
어떤 종류의 결정이 이른 결정일까? 바로 시스템의 업무 요구사항, 즉 유스케이스와 아무런 관련이 없는 결정이다. 프레임워크, 데이터베이스, 웨 서버, 유틸리티 라이브러리, 의존성 주입에 대한 결정 등이 여기 포함된다. 좋은 시스템 아키텍처란 이러한 결정이 부수적이며, 결정을 연기할 수 있는 아키텍처다. 좋은 시스템 아키텍처는 이런 결정에 의존하지 않는다. 좋은 시스템 아키텍처는 이러한 결정을 가능한 한 최후의 순간에 내릴 수 있게 해주며, 결정에 따른 영향이 크지 않게 만든다.  

### 플러그인 아키텍처
소프트웨어 개발 기술의 역사는 플러그인을 손쉽게 생성하여, 확장가능하며 유지보수가 쉬운 시스템 아키텍처를 확립할 수 있게 만드는 방법에 대한 이야기다. 선택적이거나 또는 수많은 다양한 형태로 구현될 수 있는 나머지 컴포넌트로부터 핵심적인 업무 규칙은 분리되어있고, 또한 독립적이다.  
이 설계에서 사용자 인터페이스와 데이터 베이스는 플러그인 형태로 고려되었기에, 수많은 종류의 사용자 인터페이스와 데이터베이스를 플러그인 형태로 연결할 수 있게 된다.  

### 플러그인에 대한 논의
우리는 특정 모듈이 나머지 모듈에 영향받지 않기를 바란다. 시스템을 플러그인 아키텍처로 배치함으로써 변경이 전파될 수 없는 방화벽을 생성할 수 있다. GUI가 업무규칙에 플러그인 형태로 연결되면 GUI에서 발생한 변경은 절대로 업무 규칙에 양향을 미칠 수 없다.  
경계는 변경의 축이 있는 지점에 그어진다. 경계의 한쪽에 위치한 컴포넌트는 경계 반대편의 컴포넌트와는 다른 속도로, 그리고 다른 이유로 변경된다.  
GUI는 업무규칙과는 다른 시점에 다른 속도로 변경되므로, 둘 사이에는 반드시 경계가 필요하다. 업무규칙은 의존성 주입 프레임워크와는 다른 시점에 그리고 다른 이유로 변경되므로, 둘 사이에도 반드시 경계가 필요하다.  
이 역시도 순전히 단일 책임 원칙에 해당한다. 단일 책임 원칙은 어디에 경계를 그어야 할지를 알려준다.

### 결론
소프트웨어 아키텍처에서 경계선을 그리려면 먼저 시스템을 컴포넌트 단위로 분할해야 한다. 일부 컴포넌트는 핵심 업무 규칙에 해당한다. 나머지 컴포넌트는 플러그인으로, 핵심 업무와는 직접적인 관련이 없지만 필수 기능을 포함한다. 그런 다음 컴포넌트 사이의 화살표가 특정 방향, 즉 핵심 업무를 향하도록 이들 컴포넌트의 소스를 배치한다.  
이는 의존성 역전 원칙과 안정된 추상화 원칙을 응용한 것임을 눈치챌 수 있어야 한다. 의존성 화살표는 저수준 세부사항에서 고수준의 추상화를 향하도록 배치된다.

## 18. 경계 해부학
시스템 아키텍처는 일련의 소프트웨어 컴포넌트와 그 컴포넌트들을 분리하는 경계에 의해 정의된다. 이러한 경계는 다양한 형태로 나타난다.

### 경계 횡단하기
적절한 위치에서 경계를 횡단하게 하는 비결은 소스 코드 의존성 관리에 있다.  
소스 코드 모듈 하나가 변경되면, 이에 의존하는 다른 소스 코드 모듈도 변경하거나, 다시 컴파일해야 해서 새로 배포해야 할지도 모르기 때문이다. 경계는 이러한 변경이 전파되는 것을 막는 방화벽을 구축하고 관리하는 수단으로써 존재한다.

### 두려운 단일체
아키텍처 경계 중에서 가장 단순하며 가장 흔한 형태는 물리적으로 엄격하게 구분되지 않는 형태다. 이 형태에서는 함수와 데이터가 단일 프로세서에서 같은 주소 공간을 공유하며 그저 나름의 규칙에 따라 분리되어 있을 뿐이다.  
배포 관점에서 보면 이는 소위 단일체라고 불리는 단일 실행 파일에 지나지 않는다. (실행 가능한 jar 파일, 단일 .exe 파일 등). 
이처럼 배포 관점에서 볼 때 단일체는 경계가 드러나지 않는다. 이러한 아키텍처는 거의 모든 경우에 특정한 동적 다형성에 의존하여 내부 의존성을 관리한다.  
가장 단순한 형태의 경계 횡단은 저수준 클라이언트에서 고수준 서비스로 향하는 함수 호출이다. 이 경우 런타임 의존성과 컴파일타임 의존성은 모두 같은 방향, 즉 저수준 컴포넌트에서 고수준 컴포넌트로 향한다.  
반대로 고수준 클라이언트가 저수준 서비스를 호출해야 한다면 동적 다형성을 사용하여 제어 흐름과는 반대 방향으로 의존성을 역전시킬 수 있다. 이렇게 하면 런타임 의존성은 컴파일타임 의존성과는 반대가 된다.  
정적 링크된 모노리틱 구조의 실행 파일이라도 이처럼 규칙적인 방식으로 구조를 분리하면 프로젝트를 개발, 테스트, 배포하는 작업에 큰 도움이 된다.  
단일체에서 컴포넌트간 통신은 매우 빠르고 값싸다. 통신은 전형적인 함수 호출에 지나지 않기 때문이다. 결과적으로, 소스 수준에서 결합이 분리되면 경계를 가로지르는 통신은 상당히 빈번할 수 있다.  
단일체를 배포하는 일은 일반적으로 컴파일과 정적 링크 작업을 수반하므로, 대체로 이러한 시스템에서 컴포넌트는 소스 코드 형태로 전달된다.  

### 배포형 컴포넌트
아키텍처의 경계가 물리적으로 드러날 수도 있는데 그 중 가장 단순한 형태는 동적 링크 라이브러리다. (.NET DLL, 자바 jar, 루비 젬 등) 컴포넌트를 이 형태로 배포하면 따로 컴파일하지 않고 곧바로 사용할 수 있다. 대신 컴포넌트는 바이너리와 같이 배포 가능한 형태로 전달된다. 이는 배포 수준 결합 분리 모드에 해당한다. 배포 작업은 단순히 이들 배포 가능한 단위를 좀더 편리한 형태로 묶는 일에 지나지 않는다. 예를 들어 WAR 파일이나 심지어 그냥 디렉터리 형태로 묶기도 한다.  
이러한 배포 과정에서만 차이가 날 뿐, 배포 수준의 컴포넌트는 단일체와 동일하다. 단일체와 마찬가지로 배포형 컴포넌트의 경계를 가로지르는 통신은 순전히 함수 호출에 지나지 않는다. 동적 링크와 런타임 로딩으로 인해 최초의 함수 호출은 오래 걸릴 수 있지만, 대체로 이들 경계를 가로지르는 통신은 매우 빈번할 것이다.

### 스레드
단일체와 배포형 컴포넌트는 모두 스레드를 활용할 수 있다. 스레드는 아키텍처 경계도 아니며 배포 단위도 아니다. 이보다 스레드는 실행 계획과 순서를 체계화하는 방법에 가깝다. 모든 스레드가 단 하나의 컴포넌트에 포함될 수도 있고, 많은 컴포넌트에 걸쳐 분산될 수도 있다.

### 로컬 프로세스
로컬 프로세스는 주로 명령행이나 그와 유사한 시스템 호출을 통해 생성된다. 로컬 프로세스들은 동일한 프로세서 또는 하나의 멀티코어 시스템에 속한 여러 프로세서들에서 실행되지만, 각각이 독립된 주소 공간에서 실행된다. 종종 공유 메모리 파티션을 사용하기도 하지만, 일반적으로는 메모리 보호를 통해 프로세스들이 메모리를 공유하지 못하게 한다.  
대개의 경우 로컬 프로세스는 소켓이나 메일박스, 메시지 큐와 같이 운영체제에서 제공하는 통신 기능을 이용하여 서로 통신한다.  
로컬 프로세스를 일종의 최상위 컴포넌트라고 생각하자. 즉, 로컬 프로세스는 컴포넌트 간 의존성을 동적 다향성을 통해 관리하는 저수준 컴포넌트로 구성된다.  
로컬 프로세스 간 분리 전략은 단일체나 바이너리 컴포넌트의 경우와 동일하다. 즉, 항상 고수준 컴포넌트를 향한다.  
로컬 프로세스 경계를 지나는 통신에는 운영체제 호출, 데이터 마샬링, 언마샬링, 프로세스 간 문맥 교환 등이 있으며, 이들은 제법 비싼 작업에 속한다. 따라서 통신이 너무 빈번하게 이뤄지지 않도록 신중하게 제한해야 한다.

### 서비스
물리적인 형태를 띠는 가장 강력한 경계는 바로 서비스다. 서비스는 프로세스로, 일반적으로 명령행 또는 그와 동등한 시스템 호출을 통해 구동된다. 서비스는 자신의 물리적 위치에 구애받지 않는다. 서비스들은 모든 통신이 네트워크를 통해 이뤄진다고 가정한다.  
서비스 경계를 지나는 통신은 함수 호출에 비해 매우 느리다. 따라서 가능하다면 빈번하게 통신하는 일을 피해야 한다. 이 수준의 통신에서는 지연에 따른 문제를 고수준에서 처리할 수 있어야 한다.  
이를 제외하고는 로컬 프로세스에 적용한 규칙이 서비스에도 그대로 적용된다. 저수준 서비스는 반드시 고수준 서비스에 '플러그인'되어야 한다.  

### 결론
단일체를 제외한 대다수의 시스템은 한 가지 이상의 경계 전략을 사용한다. 서비스 경계를 활용하는 시스템이라면 로컬 프로세스 경계도 일부 포함하고 있을 수 있다.  
즉, 대체로 한 시스템 안에서도 통신이 빈번한 로컬 경계와 지연을 중요하게 고려해야 하는 경계가 혼합되어 있음을 의미한다. 

## 19. 정책과 수준





