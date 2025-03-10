# Chapter 05. 스프링 컨텍스트: 빈의 스코프 및 수명 주기

- 스프링이 언제 어떻게 빈을 생성하는가?
- 스프링에는 빈을 생성하고 수명 주기를 관리하는 여러 가지 방식이 있다. 스프링 세계에서는 이런 접근 방식을 스코프(scope)라고 한다.
- 두 가지 스코프, 싱글톤과 프로토타입을 설명한다.

## 5.1 싱글톤 빈 스코프 사용

- 싱글톤 빈의 인스턴스를 생성하는 두 가지 방식, 즉시(eager) 및 지연(lazy).

### 5.1.1 싱글톤 빈의 작동 방식

- 싱글톤은 스프링에서 가장 많이 사용되는 기본 빈 스코프이다.
- 스프링은 컨텍스트를 로드할 때 싱글톤 빈을 생성하고 빈에 이름(빈 ID라고도 함.)을 할당한다.
- 스프링에서 싱글톤 개념은 동일한 타입의 여러 인스턴스를 허용하며, 싱글톤은 이름별로 고유하지만 앱 단위로 고유하지 않다는 것을 의미한다.

### 5.1.2 실제 시나리오의 싱글톤 빈

- 싱글톤 빈의 스코프는 앱의 여러 컴포넌트가 하나의 객체 인스턴스를 공유할 수 있다고 가정하기 때문에 가장 중요하게 고려해야할 점은 이런 빈이 불변(immutable)이어야 한다는 것이다.
- 실제 앱은 여러 스레드로 작업들을 실행한다. 이 때 여러 스레드는 동일한 인스턴스를 공유한다. 이런 스레드가 인스턴스를 변경하면 경쟁 상태(race condition) 시나리오가 발생한다.
- 속성이 변경되는 가변(mutable) 싱글톤 빈을 사용하려면 주로 스레드 동기화를 사용하여 동시성이 있는(concurrent) 빈이 되도록 직접 만들어야 한다.

- 스프링 컨텍스트에서 객체 빈을 만들어야 한다면, 불변인 경우에만 싱글톤으로 만들어야 한다. 변경 가능한 싱글톤 빈을 설계하지 마라.

### 5.1.3 즉시 및 지연 인스턴스 생성 방식

1. 즉시 인스턴스 생성 방식(eager instantiation)
    - 스프링이 컨텍스트를 초기화할 때 모든 싱글톤 빈을 생성하는 방식.

2. 지연 인스턴스 생성 방식(lazy instantiation)
   - 스프링이 컨텍스트를 초기화할 때 싱글톤 인스턴스를 생성하지 않고, 빈을 처음 참조할 때 각 인스턴스를 생성하는 방식.

> ### 언제 즉시 인스턴스를 사용하고 언제 지연 인스턴스를 사용할까?
> - 기본적으로 즉시 인스턴스 생성 방식을 사용하라. 일반적으로 이 방식이 장점이 더 많다.
> - 스프링 컨텍스트와 함께 모든 빈의 인스턴스를 생성하는 것이 불필요하게 많은 메모리를 차지하는 상황이라면 지연 인스턴스 생성 방식을 고려하라.
> - 아키텍처에 따라 장점이 있는 방식을 선택하라.

## 5.2 프로토타입 빈 스코프 사용

### 5.2.1 프로토타입 빈의 동작 방식

- 프로토타입 스코프의 빈에 대한 참조를 요청할 때마다 스프링은 새로운 인스턴스를 생성한다.
- 빈의 스코프를 변경하려면 @Scope라는 어노테이션을 사용해야 한다.
- 프로토타입 빈을 사용하면 빈을 요청하는 각각의 스레드가 서로 다른 인스턴스를 얻기 때문에 더 이상 동시성 문제가 발생하지 않는다. 따라서 변경 가능한(mutable) 프로토타입 빈을 정의하는 것은 문제가 되지 않는다.

### 5.2.2 실제 시나리오에서 프로토타입 빈 관리

> 어떤 객체를 빈으로 만들지 결정하기 전에 '꼭 빈이어야 할까?'라는 질문을 스스로에게 던지는 것이 중요하다.

- 일반적으로 프로토타입 빈과 가변 인스턴스를 피하는 것이 좋다.

## 5.3 요약

- 스프링에서 빈의 스코프는 프레임워크가 객체 인스턴스를 관리하는 방법을 정의한다.
- 스프링은 싱글톤과 프로토타입이라는 두 가지 빈 스코프를 제공한다.
  - 싱글톤을 사용하면 스프링은 해당 컨텍스트에서 직접 인스턴스를 관리한다. 각 인스턴스에는 고유한 이름이 있으며, 이 이름을 사용하여 항상 특정 인스턴스를 참조한다.
  - 프로토타입을 사용하면 스프링은 객체 타입만 고려한다. 각 타입에는 고유한 이름이 있다. 스프링은 빈 이름을 참조할 때마다 해당 타입의 새로운 인스턴스를 생성한다.
- 스프링이 싱글톤 빈을 생성하는 두 시점(스프링 컨텍스트가 초기화될 때(eager) 또는 빈이 첫 번째 참조될 때(lazy))을 설정할 수 있다.
- 앱에서는 싱글톤 빈을 주로 사용한다. 동이란 이름으로 참조하는 모든 곳에서 동일한 인스턴스를 얻기 때문에 여러 스레드가 이 인스턴스를 액세스하여 사용할 수 있다. 따라서 인스턴스는 불변으로 만든다.
- 빈처럼 변경 가능한 객체가 필요하면 프로토타입 스코프를 사용하는 것이 좋은 방법이 될 수 있다.
- 프로토타입 스코프의 빈을 싱글톤 스코프의 빈에 주입할 때는 주의해야 한다. 이렇게 하면 싱글톤 인스턴스는 항상 동일한 프로토타입 인스턴스를 사용하게 된다. 스프링이 싱글톤 인스턴스를 생성할 시점에 동일한 프로토타입 인스턴스를 주입하기 때문이다. 이는 일반적으로 악성 설계다. 빈을 프로토타입 스코프로 만드는 이유는 사용할 때 항상 독립된 인스턴스를 얻어야 하기 때문이다.