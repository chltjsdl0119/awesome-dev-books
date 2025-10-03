# SecurityBuilder / SecurityConfigurer

- SecurityBuilder는 빌더로서 웹 보안을 구성하는 빈 객체와 설정 클래스들을 생성하는 역할을 하는 인터페이스이다.
- SecurityConfigurer는 Http 요청과 관련된 보안 처리를 담당하는 필터들을 생성하고 여러 초기화 설정에 관여한다.


```java
public interface SecurityBuilder<O> {

	O build() throws Exception;

}
```

```java
public interface SecurityConfigurer<O, B extends SecurityBuilder<O>> {

	void init(B builder) throws Exception;
    
	void configure(B builder) throws Exception;

}
```

# HttpSecurity / WebSecurity

- FilterChainProxy 빈 객체를 만드는 것이 최종 목적이다.

## HttpSecurity

- HttpSecurityConfiguration에서 HttpSecurity 객체를 생성, 초기화한다.
- HttpSecurity는 보안에 필요한 각 설정 클래스와 필터들을 생성하고 최종적으로 SecurityFilterChain 빈 객체를 생성한다.
- 클라이언트가 보낸 요청과 SecurityFilterChain의 RequestMatcher가 매칭되면 SecurityFilterChain에 등록된 필터들이 순서대로 실행된다.

## WebSecurity

- WebSecurityConfiguration에서 WebSecurity 객체를 생성, 초기화한다.
- WebSecurity는 HttpSecurity에서 생성한 SecurityFilterChain 빈 객체를 SecurityBuilder에 등록한다.
- WebSecurity가 build() 메서드를 호출하면 SecurityBuilder에 등록된 SecurityFilterChain 빈 객체를 꺼내어 FilterChainProxy의 생성자에 전달한다.

# DelegatingFilterProxy / FilterChainProxy

## Filter

- 서블릿 필터는 웹 애플리케이션에서 클라이언트의 요청과 서버의 응답을 가공하거나 검사하는데 사용되는 구성 요소이다.
- 서블릿 필터는 클라이언트의 요청이 서블릿에 도달하기 전이나 서블릭이 응답을 클라이언트에게 보내지 전에 특정 작업을 수행할 수 있다.
- 서블릿 핏터는 서블릿 컨테이너(WAS)에서 생성되고 실행되고 종료된다.

- 필터는 Spring IoC 컨테이너에서 관리되지 않는다. 그러므로 DI, AOP 등의 기능을 사용할 수 없다.

## DelegatingFilterProxy

- DelegatingFilterProxy는 스프링에서 사용되는 특별한 서블릿 필터로, 서블릿 컨테이너와 스프링 애플리케이션 컨텍스트 간의 연결고리 역할을 하는 필터이다.
- DelegatingFilterProxy는 서블릿 필터의 기능을 수행하는 동시에 스프링의 의존성 주입 및 빈 관리 기능과 연동되도록 설계된 필터라 할 수 있다.
- DelegatingFilterProxy는 "springSecurityFilterChain"이라는 이름의 빈을 찾고, 요청을 위임한다.
- 실제 보안 처리를 수행하지 않는다.

## FilterChainProxy

- springSecurityFilterChain 이름으로 생성되는 필터 빈으로서 DelegatingFilterProxy으로부터 요청을 위임 받고 보안 처리 역할을 한다.
- 내부적으로 하나 이상의 SecurityFilterChain 객체들을 가지고 있으며, 요청 URL 정보를 기준으로 적절한 SecurityFilterChain을 선택하여 필터들을 호출한다.
- HttpSecurity를 통해 API 추가 시, 관련 필터들이 추가된다.
- 사용자의 요청을 필터 순서대로 호출함으로 보안 기능을 동작시키고 필요 시, 직접 필터를 생성해서 기존의 필터 전 또는 후에 추가할 수 있다.
