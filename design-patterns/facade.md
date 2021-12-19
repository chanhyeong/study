# FACADE (254p ~ 264p)

## Intent
한 subsystem 내의 인터페이스 집합에 대한 획일화된 interface 를 제공  
subsystem 을 사용하기 쉽도록 상위 수준의 interface 를 정의

## Motivation
복잡성을 줄이는데 도움 - subsystem 간의 의사소통 및 종속성 최소화  
주어진 subsystem 의 일반적인 기능에 관한 단순화된 하나의 interface 제공

컴파일러 시스템을 사용하는 애플리케이션은 대부분 구체적인 내용에 상관없이  
파싱 or 코드 생성 단계를 단순히 이용하기만 함 -> 코드 컴파일만 되면 끝  

컴파일러 시스템은 컴파일러 클래스만 정의 -> 제공하는 기능에 대한 인터페이스 정의 = Facade

## Applicability
- 복잡한 subsystem 에 대한 단순하면서도 기본적인 interface 를 제공하여 대부분의 개발자들에게 적합한 클래스 형태를 제공
- 호출을 단순한 형태로 통합하여 제공, 나머지는 내부적으로 처리하여 사용자 - subsystem 호출 횟수의 감소, 결합도를 줄임
- subsystem 을 계층화시킬 때 각 subsystem 에 대한 접근점 제공
    - 다른 subsystem 에 종속적이어도 제공하는 facade 를 통해서만 대화하여 종속성을 줄일 수 있음

## Structure
![image](https://user-images.githubusercontent.com/10507662/145715233-e5e5e51e-309c-47d5-a101-df56d2dd7559.png)

## Participants
- Facade (Compiler): 단순하고 일관된 통합 interface
    - subsystem 을 구성하는 어떤 클래스가 어떤 요청을 처리해야하는지 알고 있음
    - 사용자의 요청을 전달
- Subsystem Classes (Scanner, Parser, ProgramNode, etc.): subsystem 의 기능을 구현하고, Facade 객체로 할당된 작업을 실제로 처리하지만, Facade 에 대한 아무 정보도 없음. 어떤 reference 도 없음

## Collaboration
- Client 는 Facace 에 정의된 interface 를 이용하여 subsystem 과 상호작용
    - Facade 는 subsystem 을 구성하는 적당한 객체에게 전달
    - subsystem 을 구성하는 객체가 요청 처리 담당하지만 Facade 는 subsystem 에게 메시지를 전달하기 위해 자신의 interface 에 동일한 작업을 정의해야 함
- Facade 를 사용하는 Client 는 subsystem 을 구성하는 객체로 직접 접근하지 않아도 됨

## Consequences
1. subsystem 의 구성요소를 보호할 수 있음
    - Client 가 다를 객체 수가 줄고, subsystem 을 쉽게 사용할 수 있음
2. subsystem - Client code 의 결합도를 낮춤
    - 컴파일 의존성을 줄일 수 있다  
        - facade 를 사용하여 컴파일 의존성 최소화 -> subsystem 에서 작은 변경으로 들어가는 재컴파일을 제한  
    - 다른 플랫폼으로 이식도 단순해짐
        - 하나의 subsystem 을 빌드하면서 다른 모든 subsystem 까지 끌고 들어갈 때가 적어지기 때문
3. 애플리케이션에서 subsystem 클래스를 사용하는 것을 막지는 않음

## Implementations
구현하기 위해 고려할 사항들

1. Client - subsystem 간 결합도 줄이기
    - Facade: abstract class
    - subsystem: inherited subclass
    - subclassing 을 하지 않으려면 다른 subsystem 객체를 조합하여 구성할 수도 있음 (?)
2. subsystem 클래스 중 공개할 것과 감출 것을 결정하기

## Sample codes

```java
public class Client {
  // Running the Client class as application.
  public static void main(String[] args) {
    // Creating a facade for a subsystem.
    Facade facade = new Facade1
      (new Class1(), new Class2(), new Class3());
    // Working through the facade.
    System.out.println(facade.operation());
  }
}

public abstract class Facade {
  public abstract String operation();
}

public class Facade1 extends Facade {
  private Class1 object1;
  private Class2 object2;
  private Class3 object3;
 
  public Facade1(Class1 object1, Class2 object2, Class3 object3) {
    this.object1 = object1;
    this.object2 = object2;
    this.object3 = object3;
  }

  public String operation() {
    return "Facade forwards to ... "
      object1.operation1()
      object2.operation2()
      object3.operation3();
  }
}

public class Class1 {
  public String operation1() {
    return "Class1 ";
  }
}

public class Class2 {
  public String operation2() {
    return "Class2 ";
  }
}

public class Class3 {
  public String operation3() {
    return "Class3 ";
  }
}
```

## Related Patterns
- Abstract factory
    - subsystem 에 독립적인 방법, susbsytem 객체를 생성하는 interface 를 제공하기 위해 Facade 와 함께 사용할 수 있음
    - Abstract factory 는 Facade 에 대한 대안, 플랫폼에 종속적인 클래스를 감추는데 사용
- Mediator
    - 기존에 존재하는 클래스의 기능성을 추상화한다는 점에서 비슷
    - 이거의 목적은 여러 객체들 사이의 협력 관계를 추상화, 기능성의 집중화를 막자
    - 이 패턴에 참여하는 개체는 서로를 직접 알지 못하고, Mediator 를 통해서만 상호작용
    - Facade 는 interface 자체를 추상화하여 사용을 용이하게 하려는 목적
- 근데 최신버전에서는 위에 2개는 없고 이 내용만 있다
    - `Adapter - Bridge - Composite - Decorator - Facade - Flyweight - Proxy`
These patterns are classified as structural design patterns. 

## 번외 - Slf4j 는 어떻게 구현체를 찾는가
http://www.slf4j.org/manual.html#swapping