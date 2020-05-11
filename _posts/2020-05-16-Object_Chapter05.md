## 책임 중심의 설계
1. 데이터보다 행동을 먼저 결정하라
2. 협력이라는 문맥 안에서 책임을 결정하라

### 메시지 기반의 설계 관점
- "메시지를 전송해야 하는데 누구에게 전송해야 하지?"
- 클라이언트는 어떤 객체가 메시지를 수신할지 알지 못한다.
- 클라이언트는 단지 임의의 객체가 메시지를 수신할 것이라는 사실을 믿고 자신의 의도를 표현한 메시지를 전송한다.
- 그리고 메시지를 수신하기로 결정된 객체는 메시지를 처리할 '책임'을 할당 받게 된다.
  - 메시지 전송자의 관점에서는 메시지 수신자가 깔끔하게 캡슐화되는 것이다.
  - 캡슐화의 원칙을 지키기가 수월함
  - 올바른 객체지향의 설계는 클라이언트가 전송할 메시지를 결정한 후에야 비로소 객체의 상태를 저장하는 데 필요한 내부 데이터에 대해 고민하기 시작한다.

## GRASP 패턴
1. Information Expert(정보 전문가)
- 책임을 수행하는 데 필요한 정보를 가지고 있는 객체에게 할당함
2. Creator
- 생성되는 객체의 Context를 알고 있는 다른 객체가 있다면, Context를 알고 있는 객체에 해당 객체를 생성할 책임을 부여하자.
- A의 생성을 B의 역할로 부여하는 경우
  - B 객체가 A 객체를 포함하거나 참조한다.
  - B 객체가 A 객체의 정보를 기록하고 있다.
  - A 객체가 B 객체의 일부이다.
  - B 객체가 A 객체를 긴밀하게 사용하고 있다.
  - B 객체가 A 객체의 생성에 필요한 정보를 가지고 있다.
3. Controller
- 직접적으로 각 객체에 접근하지 않도록 시스템 이벤트(사용자의 요청)를 처리할 객체를 만들자.
- 서브시스템으로 들어오는 외부요청을 처리하는 객체를 만들어 사용하라.
4. Low Coupling
- 객체들 간, 서브 시스템들 간의 의존도가 낮게 역할을 부여하자.
5. High Cohesion
- 각 객체가 밀접하게 연관된 역할들만 가지도록 역할을 부여하자.
- 한 객체, 한 서브시스템이 자기 자신이 부여받은 역할만을 수행하도록 짜임새 있게 구성되어야 한다.
6. Polymorphism
- 객체의 타입에 따라 변하는 로직이 있을 때 타입을 명시적으로 정의하고 각 타입에 다형적으로 행동하는 책임을 할당한다.
7. Pure Fabrication
- 시스템 전반적으로 사용되고 있는 기능을 제공하는 역할을 한 곳으로 모아서 가상의 객체, 서브시스템을 만들어라.
- 기능들이 시스템 전반적으로 사용되고 있어 각 객체에 그 기능을 부여하는 것이 각 객체들이 특정 데이터베이스에 종속을 가져오거나, 로그를 기록하는 매커니즘을 수정할 경우 모든 객체를 수정하는 결과를 가져온다.
- Information Expert 패턴을 적용할 때 Low Coupling, High Cohesion의 원칙이 깨질 때 사용
8. Indirection
- 두 객체 사이의 직접적인 Coupling을 피하기 위해 그 사이에 다른 객체를 사용한다.
9. Protected Variations
- 변경될 여지가 있는 곳에 안정된 인터페이스를 정의해서 사용한다.
- ex) JDBC에서 DB의 변경에도 Driver의 로딩만 바꾸어도 안정된 JDBC 인터페이스를 사용할 수 있다.

# 남은 잔업
1. 데이터 중심적 설계 코드를 교재를 보며 천천히 의미를 부여해보며 다시 개선해보기
2. GRASP 패턴이 설계 개선에 잘 반영되도록 더 잘 이해하기