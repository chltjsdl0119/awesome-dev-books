# Chapter 02. 스프링의 개념과 REST API

## 스프링 패턴과 패러다임 이해하기

### IoC란

- 프레임워크나 컴포넌트 같은 외부 소스가 프로그램의 제어 흐름을 결정하도록 바꾸는 방법.

### DI란

- 프레임워크가 설정에 따라 연결 객체를 만들고 런타임에 그 객체를 프로그램에 할당한다.
- 프레임워크는 실제로 런타임에 연결 객체를 주입하는데, 이를 DI라 한다.

> DI는 IoC의 한 타입이다. IoC 컨테이너는 구현 객체를 만들고 유지 관리한다.

### AOP란

- 프로그래밍 모델에서는 종종 하나 이상의 클래스에 걸쳐 있는 기능이나 함수가 필요하다.
- 로깅, 보안, 트랜잭션 관리, 메트릭과 같은 기능이 그 예이다.
- 이 기능들은 객체 모델의 여러 지점을 가로지르는 **횡단 관심사(cross-cutting concerns**다.

> AOP를 사용하면 다음과 같은 것을 할 수 있다.
> - 횡단 관심사를 추상화하고 캡슐화한다.
> - 코드의 여러 부분에 걸쳐 관점 동작을 추가한다.
> - 코드를 쉽게 유지하고 확장할 수 있도록 횡단 관심사에 대한 코드를 모듈화한다.

## IoC 컨테이너 이해하기

- Bean의 라이프사이클을 담당하는 컨테이너.
- Bean은 다른 객체의 작동을 요구하는 의존성을 가진다.
- IoC 컨테이너는 이런 Bean을 생성할 때 객체의 의존성을 주입하는 역할을 한다.

> IoC 컨테이너와 관련된 핵심사항. IoC 컨테이너의 기반을 제공하는 중요한 인터페이스 두 개.
> 1. BeanFactory
>   - 설정 프레임워크와 기본 기능을 제공한고 Bean 인스턴스화와 연결을 처리한다.
> 2. ApplicationContext
>    - Bean 인스턴스화와 연결을 처리할 수 있으며, 엔터프라이즈에 특화된 기능들도 함께 제공한다.

- 설정 메타데이터를 사용하면 응용프로그램 객체와 타 객체 간의 상호 의존성을 표현할 수 있다.
- 설정 메타데이터는 XML 설정, 자바 어노테이션, 자바 코드의 세 가지 방법으로 나타낼 수 있다.

## AOP용 코드 작성

```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TimeMonitor {

}

// --------------------

@Aspect
@Component
public class TimeMonitorAspect {

  @Around("@annotation(TimeMonitor)")
  public Object logTime(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("Starting " + joinPoint.getSignature().toShortString());
    System.out.println(joinPoint.getSignature().getName() + " input -> " + Arrays.toString((int[])joinPoint.getArgs()[0]));
    long start = System.nanoTime();
    Object proceed = joinPoint.proceed();
    long executionTime = System.nanoTime() - start;
    System.out.println(joinPoint.getSignature().getName() + "ed -> " + Arrays.toString((int[])joinPoint.getArgs()[0]));
    System.out.println(joinPoint.getSignature().getName() + " took: " + executionTime + " ns\n");
    return proceed;
  }
}

```

## 디스패처 서블릿의 중요성 이해

- 디스패처 서블릿은 스프링 MVC의 일부분으로 프론트 컨트롤러 역할을 한다.