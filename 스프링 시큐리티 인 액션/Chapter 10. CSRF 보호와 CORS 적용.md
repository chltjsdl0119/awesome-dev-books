# Chapter 10. CSRF 보호와 CORS 적용

## 10.1 애플리케이션에 CSRF(사이트 간 요청 위조) 보호 적용

### 10.1.1 스프링 시큐리티의 CSRF 보호가 작동하는 방식

- CSRF 보호의 시작점은 필터 체인의 CsrfFilter라는 한 필터다.
- CsrfFilter는 요청을 가로채고 GET, HEAD, TRACE, OPTION을 포함하는 HTTP 방식의 요청을 모두 허용하고 다른 모든 요청에는 토큰이 포함된 헤더가 있는지 확인한다.
- 여기서 말하는 토큰은 하나의 문자열 값이며, 사용자가 처음 웹 페이지를 열 때 서버가 CSRF 토큰을 생성한다.

#### 컨트롤러 클래스
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String getHello() {
        return "Get Hello!";
    }

    @PostMapping("/hello")
    public String postHello() {
        return "Post Hello!";
    }
}
```

#### 맞춤형 필터 클래스의 정의
```java
@Slf4j
public class CsrfTokenFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        Object o = servletRequest.getAttribute("_csrf");
        CsrfToken token = (CsrfToken) o;

        log.info(token.getToken());

        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

#### 구성 클래스에 맞춤형 필터 추가
```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .addFilterAfter(new CsrfTokenFilter(), CsrfFilter.class)
                .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
                ;

        return http.build();
    }
```

### 10.1.2 실제 시나리오 에서 CSRF 보호 사용

- CSRF 보호는 브라우저에서 실행되는 웹 앱에 이용되며, 앱의 표시된 컨텐츠를 로드하는 브라우저가 변경 작업을 수행할 수 있다고 예상될 때 필요하다.
- CSRF 토큰은 같은 서버가 프론트엔드와 백엔드 모두를 담당하는 단순한 아키텍처에서 잘 작동한다.

### 10.1.3 CSRF 맞춤 보호 구성

- CSRF 보호는 서버에서 생성된 리소스를 이용하는 페이지가 같은 서버에서 생성된 경우에만 이용한다.

#### 컨트롤러 클래스
```java
@RestController
public class HelloController {

    @PostMapping("/hello") // CSRF 보호
    public String postHello() {
        return "Post Hello!";
    }

    @PostMapping("/ciao") // CSRF 보호 해제
    public String postCial() {
        return "Post Ciao!";
    }
}
```

#### CSRF 보호를 구성하는 Customizer 객체
```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.ignoringRequestMatchers("/ciao"))
                .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
                ;

        return http.build();
    }
}
```

- 스프링 시큐리티는 DefaultCsrfToken이라는 구현을 제공한다.


- 애플리케이션이 토큰을 관리하는 방법을 변경하려면 CsrfTokenRepository 인터페이스를 구현해 맞춤형 구현을 프레임워크에 연결해야 한다.

#### CsrfTokenRepository 인터페이스의 구현
```java
@Repository
@RequiredArgsConstructor
public class CustomCsrfTokenRepository implements CsrfTokenRepository {

    private final JpaTokenRepository jpaTokenRepository;

    @Override
    public CsrfToken generateToken(HttpServletRequest request) {
        String uuid = UUID.randomUUID().toString();
        return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", uuid);
    }

    @Override
    public void saveToken(CsrfToken csrfToken, HttpServletRequest request, HttpServletResponse response) {
        String identifier = request.getHeader("X-IDENTIFIER");
        Optional<Token> existingToken = jpaTokenRepository.findTokenByIdentifier(identifier);

        if (existingToken.isPresent()) {
            Token token = existingToken.get();
            token.setToken(csrfToken.getToken());
        } else {
            Token token = new Token();
            token.setToken(csrfToken.getToken());
            token.setIdentifier(identifier);
            jpaTokenRepository.save(token);
        }
    }

    @Override
    public CsrfToken loadToken(HttpServletRequest request) {
        String identifier = request.getHeader("X-IDENTIFIER");

        Optional<Token> existingToken = jpaTokenRepository.findTokenByIdentifier(identifier);

        if (existingToken.isPresent()) {
            Token token = existingToken.get();
            return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", token.getToken());
        }

        return null;
    }
}
```

#### CustomCsrfTokenRepository 구성 클래스에 설정하기
```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.csrfTokenRepository(new CustomCsrfTokenRepository()))
                .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
                ;

        return http.build();
    }
}
```

## 10.2 CORS 이용

### 10.2.1 CORS 작동 방식

- 애플리케이션이 두 개의 서로 다른 도메인 간에 호출하는 것은 모두 금기된다. 그러나 그러한 호출이 필요할 때가 있는데, 이 때 CORS를 이용하면 된다.
- CORS 메커니즘은 HTTP 헤더를 기반으로 작동한다.

> CORS 메커니즘의 중요한 헤더
> - Access-Control-Allow-Origin: 도메인의 리소스에 접근할 수 있는 외부 도메인을 지정.
> - Access-Control-Allow-Methods: 특정 HTTP 방식만 허용하고 싶을 때, 방식 지정 가능.
> - Access-Control-Allow-Headers: 특정 요청에 이용할 수 있는 헤더에 제한을 추가

- CORS는 제한을 가하기보다 교차 도메인 호출의 엄격한 제약 조건을 완화하도록 도와주는 기능이다.
- 종종 브라우저는 요청을 허용해야 하는지 테스트하기 위해 먼저 HTTP OPTIONS 방식으로 호출하는 경우가 있는데, 이 테스트 요청을 사전(Preflight) 요청이라 한다. 이 요청이 실패하면 브라우저는 원래 요청을 수락하지 않는다.

### 10.2.2 @CrossOrigin 어노테이션으로 CORS 정책 적용

- 엔드포인트를 정의하는 메서드 바로 위에 @CrossOrigin 어노테이션을 배치하고 허용된 출처와 메서드를 이용해 구성할 수 있다.

#### localhost가 출처를 허용하도록 지정
```java
    @PostMapping("/test")
    @ResponseBody
    @CrossOrigin("http://localhost:8080")
    public String test() {
        return "Hello";
    }
```

- @CrossOrigin의 값 매개 변수는 여러 출처를 정의하는 배열을 받는다.
- 모든 출처를 허용하면 애플리케이션이 XSS 요청에 노출되고 결과적으로 DDoS 공격에 취약해질 수 있다.

### 10.2.3 CorsConfigurer로 CORS 적용

#### 구성 클래스로 중앙 집중된 CORS 구성 정의
```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .cors(cors -> {
                    CorsConfigurationSource source = request -> {
                        CorsConfiguration config = new CorsConfiguration();
                        config.setAllowedOrigins(List.of("example.com", "example.org"));
                        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
                        return config;
                    };

                    cors.configurationSource(source);
                })
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
                ;

        return http.build();
    }
}
```

- HttpSecurity 객체에서 호출하는 cors() 메서드는 Customizer<CorsConfigurer> 객체를 매개 변수로 받는다.
- 위 예제는 간단한 예제이다. 실제 애플리케이션에서는 CorsConfigurationSource를 다른 클래스로 나누는 것이 좋다.

## 요약
- CSRF는 사용자를 속여 위조 스크립트가 포함된 페이지에 접근하도록 하는 공격 유형이다.
- CSRF 보호는 스프링 시큐리티에서 기본적으로 활성화된다.
- 스프링 시큐리티 아키텍처에서 CSRF 보호 논리의 진입점은 HTTP 필터다.
- CORS는 특정 도메인에서 호스팅되는 웹 애플리케이션이 다른 도메인의 컨텐츠에 접근하려고 할 때 발생하며 기본적으로 브라우저는 이러한 접근을 허용하지 않는다. CORS 구성을 이용하면 리소스의 일부를 브라우저에서 실행되는 웹 애플리케이션의 다른 도메인에서 호출할 수 있다.
- CORS를 구성하는 방법에는 @CrossOrigin 어노테이션으로 엔드포인트별로 구성하는 방법과 HttpSecurity 객체의 cors() 메서드로 중앙화된 구성 클래스에서 구성하는 방법이 있다.
