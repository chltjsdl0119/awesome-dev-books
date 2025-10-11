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
