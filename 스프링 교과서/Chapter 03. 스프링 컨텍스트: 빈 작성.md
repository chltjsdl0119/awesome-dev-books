# Chapter 03. 스프링 컨텍스트: 빈 작성

- 스프링은 IoC 원칙을 사용하기 대문에 어떤 앱 객체를 제어해야 할지 스프링에 알려야 한다.
- 객체지향 프로그래밍 언어에서는 객체가 동작을 구현할 때, 다른 객체에 특정 책임을 위임해야 할 때가 많다.

> 빈 간 관계를 성정하는 두 가지 방법
> - 빈을 생성하는 메서드를 직접 노출하여 빈을 연결하는 방법.(와이어링)
> - 스프링이 메서드 매개변수를 이용하야 값을 제공하도록 활성화하는 방법.(오토 와이어링)

## 3.1 구성 파일에서 정의된 빈 간 관계 구현

### 3.1.1 두 @Bean 메서드 간 직접 메서드를 호출하는 빈 작성

```java
// 직접 메서드 호출을 사용하는 빈 간 링크 설정하기(와이어링)

@Configuration
public class ProjectConfig {
    
    @Bean
    public parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }
    
    @Bean
    public person person() {
        Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot()); // Parrot 인스턴스가 두 개 생성되는 것이 아닌 앵무새 빈을 참조한다는 것.
        return p;
    }
}
```

### 3.1.2 @Bean 메서드의 매개변수로 빈 와이어링하기

```java
// 메서드의 매개변수를 사용하여 빈 의존성 주입하기

@Configuration
public class ProjectConfig {
    
    @Bean
    public parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }
    
    @Bean
    public person person(Parrot parrot) { // 스프링은 이 매개변수에 앵무새 빈을 주입한다.
        Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot());
        return p;
    }
}
```

- 여기서 주입은 DI, 즉 의존성 주입이다.
- DI는 프레임워크가 특정 필드 또는 매개변수에 값을 설정하는 기법이다.
- DI는 IoC 원리를 응용한 것이다.
- DI는 생성된 인스턴스를 관리하고 앱을 개발할 때 작성하는 코드를 최소화하는 데 도움이 되는 매우 편리한 방법이다.

## 3.2 @Autowired 어노테이션을 사용한 빈 주입

>@Autowired 어노테이션을 사용하는 방법 세 가지
> - 클래스의 필드에 값 주입하기: 예제와 개념 증명(PoC)에서 흔히 볼 수 있는 방법.
> - 클래스의 생성자 매개변수로 값 주입하기: 실제 시나리오에서 가장 자주 사용하는 방법.
> - setter(설정자)로 값 주입하기: 프로덕션 수준의 코드에서 거의 사용하지 않는 방법.

### 3.2.1 @Autowired로 클래스 필드를 이용한 값 주입

- 매우 간단하지만 단점이 존재해 프로덕션 코드를 작성할 때는 사용하지 않는다. 하지만 예제, 개념 증명(PoC), 테스트 작성에서 자주 쓴다.

```java
@Component
public class Person {
    
    private String name = "Ella";
    
    @Autowired // 필드에 이 어노테이션을 추가하면, 해당 컨텍스트에서 적절한 값을 주입하도록 스프링에 지시한다.
    private Parrot parrot;
    
    // getter, setter
}
```

#### 프로덕션 코드에서 바람직하지 않은 이유

- 필드를 final 필드로 만들 수 있는 방법이 없다.
- 초기화할 때 값을 직접 관리하는 것이 더 어렵다.

### 3.2.2 @Autowired를 사용하여 생성자로 값 주입

- 프로덕션 코드에서 가장 자주 이용되는 방식이다.
- 이 방법을 이용할 시, 필드를 final 로 정의할 수 있어 스프링이 필드를 초기화한 후에는 아무도 필드 값을 변경할 수 없다.

```java
@Component
public class Person {
    
    private String name = "Ella";
    private final Parrot parrot;

    @Autowired 
    public Person(Parrot parrot) { // 스프링이 Person 타입 빈을 생성할 때, @Autowired 가 달린 이 생성자를 호출한다.
        this.parrot = parrot;
    }
    
    // getter, setter
}
```

- 스프링 4.3 버전부터 클래스에 생성자가 하나 있다면 @Autowired 어노테이션을 생략할 수 있다.

### 3.2.3 setter를 이용한 의존성 주입 사용

- 가독성이 떨어지고, final 필드를 만들 수 없으며, 테스트를 더 쉽게 만드는 데 도움이 되지 않는 등의 단점이 많아 잘 사용되지 않는다.

```java
@Component
public class Person {
    
    private String name = "Ella";
    private final Parrot parrot;

    @Autowired 
    public void setParrot(Parrot parrot) {
        this.parrot = parrot;
    }
    
    // getter, setter
}
```

## 3.3 순환 의존성 다루기

> 순환 의존성(circular dependency)
> - 빈 A 생성을 위해 스프링이 아직 없는 다른 빈 B 를 주입해야하는 상황을 의미한다.
> - 두 빈은 서로에 대한 의존성이 필요하다.
> - 이로 인해 스프링은 교착 상태에 빠진다.
> - 피하는 방법: 생성을 위해 다른 객체에 의존해야 하는 객체를 정의하지 않는지 확인하면 된다.

## 3.4 스프링 컨텍스트에서 여러 빈 중 선택하기

- 스프링 컨텍스트에 동일한 타입의 빈이 여러 개 있을 경우 어떤 빈을 주입할까?

```java
// 매개변수명으로 주입하기

@Configuration
public class ProjectConfig {
    
    @Bean
    public Person person(Parrot parrot2) { // 매개변수 이름이 parrot2와 일치하는 빈을 주입한다.
        Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot2);
        return p;
    }
}
```

```java
// @Qualifier 어노테이션 사용으로 주입하기

@Configuration
public class ProjectConfig {
    
    @Bean
    public Person person(@Qualifier("Parrot2") Parrot parrot) { // 매개변수 이름이 parrot2와 일치하는 빈을 주입한다.
        Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot2);
        return p;
    }
}
```

## 3.5 요약

- 스프링 컨텍스트는 프레임워크가 관리하는 객체를 유지하는 데 사용하는 앱 메모리 공간이다. 스프링 컨텍스트에서 보강해야 하는 모든 객체를 추가해야 한다.
- 앱을 구현할 때는 객체에서 다른 객체를 참조해야 한다. 이것으로 한 객체가 자신의 책임을 실행할 때 다른 객체에 작업을 위임할 수 있다. 이 동작을 구현하기 위해선 스프링 컨텍스트에서 빈 간 관계를 설정해야 한다.
- 빈 간 관계 설정 방법
  1. 빈을 생성하는 메서드에서 다른 빈을 생성하는 @Bean 어노테이션된 메서드를 직접 참조한다.
  2. @Bean 어노테이션된 메서드에 매개변수를 정의한다.
  3. @Autowired 어노테이션을 사용한다.
- @Autowired 사용법
  1. 필드 주입: 예제와 개념 증명(PoC)에 자주 사용된다.
  2. 생성자 주입: 가장 많이 사용된다.
  3. setter 주입: 자주 사용되지 않는다.
- 스프링에서 클래스의 속성이나 메서드 또는 생성자 매개변수를 사용하여 값이나 참조를 제공하도록 허용할 때는 스프링이 IoC 원칙으로 지원되는 기술인 DI를 사용한다고 할 수 있다.
- 서로 의존하는 빈 두 개를 생성하면 순환 의존성이 발생한다.
- 컨텍스트에 타입이 동일한 빈이 두 개 이상 있을 때 스프링은 어떤 빈을 주입해야 하는지 정하지 못한다. 이럴 때 빈을 주입하는 방법은 다음과 같다.
  1. @Primary 어노테이션을 사용하여 빈 중 하나를 기본값으로 설정하는 방법.
  2. @Qualifier 어노테이션을 사용하여 빈 이름으로 주입하는 방법.