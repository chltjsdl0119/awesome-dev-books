# Chapter 02. 스프링 컨텍스트: 빈 정의

- 컨텍스트는 사용자가 정의한 인스턴스를 스프링이 제어할 수 있게 해주는 복잡한 메커니즘이다.

## 2.1 메이븐 프로젝트 생성

- 빌드 도구는 앱을 더 쉽게 빌드하는 데 사용하는 소프트웨어다.

> 앱 빌드에 자주 포함되는 작업 몇 가지
> - 앱에 필요한 의존성 내려받기
> - 테스트 실행
> - 구문이 정의한 규칙 준수 여부 검증
> - 보안 취약점 확인
> - 앱 컴파일
> - 실행 가능한 아카이브에 앱 패키징

> 새 프로젝트 생성 시 고려
> - 그룸 ID(Group ID): 관련된 여러 프로젝트를 그룹화하는 데 사용
> - 아티팩트 ID(Artifact ID): 현재 애플리케이션 이름
> - 버전(Version): 현재 구현 상태의 식별자

- 메이븐은 기본적으로 'Maven central' 이라는 레포지토리에서 의존성(일반적으로 JAR 파일)을 내려받는다.

## 2.2 스프링 컨텍스트에 새로운 빈 추가

> 컨텍스트에 빈 추가하는 방법
> - @Bean 어노테이션 사용
> - 스테레오타입(stereotype) 어노테이션 사용
> - 프로그래밍 방식

### 2.2.1 @Bean 어노테이션을 사용하여 스프링 컨텍스트에 빈 추가

```java
// 스프링 컨텍스트 인스턴스 생성

public class Main {
    
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext();
        
        Parrot p = new Parrot();
    }
}
```

```java
// @Bean 메서드 정의하기

@Configuration
public class ProjectConfig {
    
    @Bean // 스프링에 컨텍스트가 초기화될 때, 이 메서드를 호출하고 반환된 값을 컨텍스트에 추가.
    Parrot parrot() { // 스프링 컨텍스트에 빈을 추가할 때, 메서드 컨벤션을 따르지 않는다.
        var p = new Parrot();
        p.setName("Koko");
        return p;
    }
}
```

```java
// 정의된 구성 클래스를 기반으로 스프링 컨텍스트 초기화하기

public class Main {
    
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class); // 스트링 컨텍스트 인스턴스가 생성될 때 구성 클래스를 매개변수로 전송하여 스프링이 이를 사용하도록 지시한다.
        
    }
}
```

- 스프링 컨텍스트의 목적을 기억하라. 스프링이 관리해야 할 것으로 생각되는 인스턴스를 추가하는 것이다.
- 빈 객체의 이름은 이를 정의하는 @Bean 메서드의 이름과 같다.

```java
// 타입으로 Parrot 인스턴스 참조하기

public class Main {
    
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        
        Parrot p = context.getBean(Parrot.class); // Parrot 인스턴스가 여러 개일 경우 예외가 발생한다. 
        System.out.println(p.getName);
        
    }
}
```

```java
// 식별자로 빈 참조하기

public class Main {
    
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        
        Parrot p = context.getBean("Parrot2", Parrot.class); // 첫 번째 매개변수가 참조할 인스턴스 이름이다. 
        System.out.println(p.getName);
        
    }
}
```

```java
// @Bean 어노테이션의 name 또는 value 속성 중 하나를 사용하여 빈의 이름 지정하기

@Configuration
public class ProjectConfig {
    
    @Bean(name = "miki") // 빈 이름을 설정한다.
    Parrot parrot() {
        var p = new Parrot();
        p.setName("miki");
        return p;
    }
}
```

```java
// @Primary 어노테이션으로 기본 빈 지정

@Configuration
public class ProjectConfig {
    
    @Bean
    @Primary // 기본 빈 지정 어노테이션
    Parrot parrot() {
        var p = new Parrot();
        p.setName("miki");
        return p;
    }
}
```

### 2.2.2 스테레오타입 어노테이션으로 스프링 컨텍스트에 빈 추가

1. @Component 어노테이션으로 스프링이 해당 컨텍스트에 인스턴스를 추가할 클래스를 표시한다.
2. 구성 클래스 위에 @ComponentScan 어노테이션으로 표시한 클래스를 어디에서 찾을 수 있는지 스프링에 지시한다.

```java
// Parrot 클래스에 대해 스테레오타입 어노테이션 사용하기

@Component // 스프링은 이 클래스의 인스턴스를 생성하고 스프링 컨텍스트에 추가한다. 하지만 @ComponentScan 어노테이션으로 지정된 클래스를 검색하도록 지시해야 한다.
public class Parrot {
    
    private String name;
    // getter, setter
}
```

```java
// @ComponentScan 어노테이션으로 스프링이 검색할 위치 지정하기

@Configuration
@ComponentScan(basePackages = "main") // main 패키지에서 검색을 한다.
public class ProjectConfig {
    
}
```
