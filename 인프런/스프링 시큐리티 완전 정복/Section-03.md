# 인증 프로세스

## 폼 인증 - formLogin()

- HTTP 기반의 폼 로그인 인증 메커니즘을 활성화하는 API로서 사용자 인증을 위한 사용자 정의 로그인 페이지를 쉽게 구현할 수 있다.
- 기본적으로스프링 시큐리티가 제공하는 기본 로그인 페이지를 사용하며 사용자 이름과 비밀번호 필드가 포함된 간단한 로그인 양식을 제공한다.
- 사용자는 웹 폼을 통해 자격 증명을 제공하고 Spring Security는 HttpServletRequest에서 이 값을 읽어 온다.

> 절차
> 1. AuthorizationFilter가 요청을 가로채고 인증이 필요한지 확인한다.
> 2. 인증이 되지 않은 대상일 시, AccessDeniedException이 발생한다.
> 3. ExceptionTranslationFilter가 AccessDeniedException을 처리하고 인증이 필요하다는 것을 인지한다.
> 4. ExceptionTranslationFilter는 AuthenticationEntryPoint를 호출하여 인증 프로세스를 시작한다.
> 5. AuthenticationEntryPoint는 로그인 페이지로 리디렉션한다.

### formLogin() 커스터마이징 예시
```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .formLogin(form -> form
                        .loginPage("/login")
                        .loginProcessingUrl("/loginProc")
                        .defaultSuccessUrl("/", true)
                        .failureUrl("/failed")
                        .usernameParameter("username")
                        .passwordParameter("password")
                        .successHandler(new AuthenticationSuccessHandler() {
                            @Override
                            public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                                System.out.println("authentication: " + authentication.getName());
                                response.sendRedirect("/home");
                            }
                        })
                        .failureHandler(new AuthenticationFailureHandler() {
                            @Override
                            public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
                                System.out.println("exception: " + exception.getMessage());
                                response.sendRedirect("/failed");
                            }
                        })
                        .permitAll()
                );

        return http.build();
    }
```

## 폼 인증 필터 UsernamePasswordAuthenticationFilter

- 스프링 시큐리티는 AbstractAuthenticationProcessingFilter 클래스를 사용자의 자격 증명을 인증하는 기본 필터로 사용한다.
- UsernamePasswordAuthenticationFilter는 AbstractAuthenticationProcessingFilter의 확장 클래스로 HttpServletRequest에서 제출된 사용자 이름과 암호를 추출하여 인증을 수행한다.
- 인증 프로세스가 초기활될 때 로그인 페이지와 로그아웃 페이지 생성을 위 DefaultLoginPageGeneratingFilter와 DefaultLogoutPageGeneratingFilter가 초기화된다.

## 기본 인증 - httpBasic()

- HTTP는 액세스 제어와 인증을 위한 프레임워크를 제공하며 가장 일반적인 인증 방식은 "Basic" 인증 방식이다.
- 클라이언트가 서버로 접속할 때 Base64로 인코딩된 사용자 이름과 비밀번호를 Authorization 헤더에 포함시켜 전송한다.

### httpBasic() API

- httpBasicConfigurer 설정 클래스를 통해 여러 API를 설정할 수 있다.
- 내부적으로 BasicAuthenticationFilter를 생성되 기본 인증 방식의 인증 처리를 담당하게 된다.

## 기본 인증 필터 - BasicAuthenticationFilter

- 이 필터는 기본 인증 서비스를 제공하는 데 사용된다.
- BasicAuthenticationFilterConverter를 사용하여 요청 헤더에 기술된 인증 정보 유효성을 체크하 Base64로 인코딩된 username과 password를 추출한다.
- 세션을 사용하는 경우 매 요청마다 인증 과정을 거치지 않으나 세션을 사용하지 않는 경우 매 요청마다 인증 과정을 거치게 된다.

## 기억하기 인증 - rememberMe()

### RememberMe 인증

- 사용자가 웹 사이트나 애플리케이션에 로그인할 때 자동으로 인증 정보를 기억하는 기능이다.
- UsernamePasswordAuthenticationFilter와 함께 사용되며, AbstractAuthenticationProcessingFilter의 확장 클래스인 RememberMeAuthenticationFilter 슈퍼 클래에서 혹은 구현된다.
  - 인증 성공 시, RememberMeServices.loginSuccess()를 통해 RememberMe 토큰을 생성하 쿠키로 전달한다.
  - 인증 실패 시, RememberMeServices.loginFail()를 통해 쿠키를 지운다.

### 토큰 생성

- 기본적으로 암호화된 토큰으로 생성 되어지며 브라우저에 쿠키를 보내고, 향후 세선에서 이 쿠키를 감지하여 자동 로그인이 이루어지는 방식으로 달성된다.

### RememberMeServices 구현체

- TokenBasedRememberMeServices - 쿠키 기반 토큰의 보안을 위해 해싱을 사용한다.
- PersistentTokenBasedRememberMeServices - 생성된 토큰을 저장하기 위해 데이터베이스나 다른 영구 저장 매체를 사용한다.
