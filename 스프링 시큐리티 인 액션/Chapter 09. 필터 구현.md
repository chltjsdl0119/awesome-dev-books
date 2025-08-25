# Chapter 09. 필터 구현

## 9.1 스프링 시큐리티 아키텍쳐의 필터 구현

- 커스텀 필터를 만드려면 Filter 인터페이스를 구현한다. doFilter() 메서드를 재정의해 논리를 구현하면 된다.


- **필터 체인**은 필터가 작동하는 순서가 정의된 필터의 모음을 나타낸다.
- 여러 필터가 같은 위치에 있으면 필터가 호출되는 순서는 정해지지 않는다.

## 9.2 체인에서 기존 필터 앞에 필터 추가

#### doFilter() 메서드에서 논리 구현
```java 
public class RequestValidationFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        var httpRequest = (HttpServletRequest) request;
        var httpResponse = (HttpServletResponse) response;

        String requestId = httpRequest.getHeader("Request-Id");

        if (requestId == null || requestId.isBlank()) {
            httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            return;
        }

        chain.doFilter(request, response);
    }
}
```

#### BasicAuthenticationFilter 앞에 필터 추가하기
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .addFilterBefore(new RequestValidationFilter(), BasicAuthenticationFilter.class);

        return http.build();
    }
}
```

## 9.3 체인에서 기존 필터 뒤에 필터 추가

#### 요청을 기록하는 필터
```java
@Slf4j
public class AuthenticationLoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        var httpRequest = (HttpServletRequest) request;

        var requestId = httpRequest.getHeader("Request-Id");

        log.info("Successfully authenticated request with id " + requestId);

        chain.doFilter(request, response);
    }
}
```

#### 필터 체인에서 기존 필터 뒤에 맞춤형 필터 추가
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .addFilterBefore(new RequestValidationFilter(), BasicAuthenticationFilter.class)
                .addFilterAfter(new AuthenticationLoggingFilter(), BasicAuthenticationFilter.class);

        return http.build();
    }
}
```

## 9.4 필터 체인의 다른 필터 위치에 필터 추가

- 기존 필터의 위치에 다른 필터를 적용하면 필터가 대체된다고 생각하는 경우가 많은데 그렇지 않다.

## 9.5 스프링 시큐리티가 제공하는 필터 구현

- 프레임워크는 필터 체인에 추가한 필터를 요청당 한 번만 실행하도록 보장하지 않는다. OnePerRequestFilter는 이름이 의미하듯이 필터의 doFilter() 메서드가 요청당 한 번만 실행되도록 논리를 구현한다.
- 애플리케이션에 이러한 기능이 필요하면 스프링이 제공하는 클래스를 이용하는 것이 좋지만 필요가 없다면 최대한 간단하게 구현할 수 있는 방법을 선택하는 것이 좋다.

#### OncePerRequestFilter 클래스 확장
```java
@Slf4j
public class AuthenticationLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        var httpRequest = (HttpServletRequest) request;

        var requestId = httpRequest.getHeader("Request-Id");

        log.info("Successfully authenticated request with id " + requestId);

        filterChain.doFilter(request, response);
    }
}
```

> OncePerRequestFilter 클래스에 관해 알아둘 사항
> - HTTP 요청만 지원하지만 사실은 항상 이것만 이용한다. Filter 인터페이스의 경우에는 요청과 응답을 형변환해야 한다.
> - 필터가 적용될지 결정하는 논리를 구현할 수 있다. 필터 체인에 추가한 필터가 특정 요청에는 적용되지 않는다고 결정할 경우, shouldNotFilter(HttpServletRequest) 메서드를 재정의하면 된다.
> - OncePerRequestFilter는 기본적으로 비동기 요청이나 오류 발송 요청에는 적용되지 않는다. 이 동작을 변경하려면 shouldNotFilterAsyncDispatch() 및 shouldNotFilterErrorDispatch() 메서드를 재정의하면 된다.

## 요약

- 웹 애플리케이션의 첫 번째 계층은 HTTP 요청을 가로채는 필터 체인이다.스프링 시큐리티 아키텍처의 다른 구성 요소는 요구사항에 맞게 맞춤 구성할 수 있다.
- 필터 체인에서 기존 필터 위치 또는 앞이나 뒤에 새 필터를 추가해 필터 체인을 맞춤 구성할 수 있다.
- 기존 필터와 같은 위치에 여러 필터를 추가할 수 있으며 이 경우 필터가 실행되는 순서는 정해지지 않는다.
- 필터 체인을 변경하면 애플리케이션의 요구 사항에 맞게 인증과 권한 부여를 맞춤 구성하는 데 도움이 된다.
