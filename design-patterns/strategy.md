# STRATEGY (407p ~ 418p)

## Intent
동일 계열의 알고리즘 군 정의, 캡슐화 -> 상호교환이 가능하게 함 (interchangeable)  
알고리즘을 사용하는 클라이언트와 상관없이 독립적으로 알고리즘을 다양하게 변경할 수 있게 함

#### 다른 이름
Policy

## Solution
(원서 내용)
![image](https://user-images.githubusercontent.com/10507662/146663388-97e34329-022e-42e6-bb9f-9907b97e9afd.png)

- 개별 Strategy 객체로 알고리즘을 캡슐화
- 알고리즘을 직접 구현하는 것 대신, 알고리즘을 대표하는 하나의 클래스로 대표함
    - 알고리즘을 수행하는 공통 인터페이스 정의 (Strategy | algorithm(…)).
    - Strategy 인터페이스를 구현하는 클래스들 정의
    - compile-time flexibility (상속을 통해)
    - run-time flexibility (객체 조합을 통해)

## Motivation
알고리즘을 직접 클래스에 하드코딩하는 것은 좋지 않음
- 프로그램이 복잡해짐 (크기 증가 및 유지보수 어려움)
- 케이스마다 필요한 알고리즘이 다름 (모든 알고리즘 제공할 필요 없음)
- 알고리즘 구성이 클라이언트 코드와 합쳐져 있는 경우, 새로운 알고리즘을 추가하거나 기존 것을 다양화하기 어려움

--> 알고리즘을 하나씩 캡슐화한 클래스 (STRATEGY) 로 정의

#### 원서 내용
Consider the right design (solution):
- 알고리즘 캡슐화
    – 알고리즘은 개별 클래스로 구현될 수 있음 (Strategy1,Strategy2,...)
    - 결과: 새로운 알고리즘 추가가 쉽고, 기존 알고리즘들을 각각 Context 클래스를 건들지 않고 변경할 수 있음
- 조건문이 필요하지 않음
    - 조건문은 다른 Strategy 객체로 대체할 수 있음
- 클래스 단순화
    - 알고리즘을 대표하는 클래스는 덜 복잡하고 빠르게 구현, 변경, 테스트, 재사용 할 수 있음

## Applicability

### 설계 관점
- runtime 에 알고리즘 교체

### 리팩토링 관점
- 하드코딩된 알고리즘 리팩토링
- 중복, 복잡한 코드 대체

## Structure
위의 solution 그림

## Participants
- Strategy: 공통 연산을 인터페이스로 정의. Context 는 ConcreteStrategy 클래스의 실제 알고리즘 사용
- ConcreteStrategy: Strategy 인터페이스를 알고리즘으로 구현
- Context: Strategy 서브클래스의 인스턴스를 갖고 구체화, Strategy 객체가 자료에 접근하는데 필요한 인터페이스 정의

## Collaboration
- Context 는 개별 Strategy 클래스를 이용하여 작업 수행
    - Context -> Strategy: 알고리즘에 필요한 데이터를 보냄
        - Context 객체 자체를 전송할 수도 있움
- Context 는 클라이언트 요청을 각 Strategy 객체로 전달
    - Client -> Context: 알고리즘에 해당하는 ConcreteStrategy 객체를 생성하여 전송

## Consequences
### Advantages
1. compile time 구현 의존성을 피할 수 있음
2. subclassing 을 사용하지 않는 유연한 대체재
3. 조건문을 없앨 수 있음

### Disadvantages
1. 공통 Strategy 인터페이스가 복잡해줄 수 있음
2. 클라이언트가 strategy 를 알아야 함
3. Strategy - Context 간 communication overhead
    - 어떤 Strategy 는 넘어온 정보를 다 사용하지 않는데도 받아야 함
4. 객체 수 증가

## Implementations
- Strategy, Context 인터페이스 정의
    - Strategy 가 Context 의 정보를 효율적으로 접근할 수 있고, 그 반대도 가능해야 함
    - 방법  - Push data: 데이터를 파라미터로 보내기
        - 결합도를 낮출 수 있음. 필요없는 데이터를 전달할 수도 있음
    - 방법  - Pull data: Context 를 Strategy argument 로 보내고 직접 꺼냄
        - Context 에 대해 정교한 인터페이스를 정의해야 함
        - 결합도가 높아짐

## Sample codes
```java
public class MyApp {
  public static void main(String[] args) {
    // Creating a Context object
    // and configuring it with a Strategy  object.
    Context context = new Context(new Strategy1());
    
    // Calling an operation on context.
    System.out.println("(1) "   context.operation());
    // Changing context's strategy.
    context.setStrategy(new Strategy2());
    System.out.println("(2) "   context.operation());
  }
}

public class Context {
  private Strategy strategy;
  
  public Context(Strategy strategy) {
    this.strategy = strategy;
  }
   
  public String operation() {
    return "Context: Delegating an algorithm to a strategy: Result = "
    strategy.algorithm();
  }

  public void setStrategy(Strategy strategy) {
    this.strategy = strategy;
  }
}

public interface Strategy {
  int algorithm();
}

public class Strategy1 implements Strategy {
  public int algorithm() {
    // Implementing the algorithm.
    return 1; // return result
  }
}

public class Strategy2 implements Strategy {
  public int algorithm() {
    // Implementing the algorithm.
    return 2; // return result
  }
}
 ```

## Related Patterns
Strategy 객체는 규묘가 작으므로 flyweight 패턴으로 정의하는 것이 좋음
