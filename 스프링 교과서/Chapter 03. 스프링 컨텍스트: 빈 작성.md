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