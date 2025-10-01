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
