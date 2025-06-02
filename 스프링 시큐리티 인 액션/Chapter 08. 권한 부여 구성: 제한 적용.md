# Chapter 08. 권한 부여 구성: 제한 적용

- anyRequest() 메서드는 이 책에서 이용한 첫 번째 선택기(matcher) 메서드다.
- antMatchers() 메서드, mvcMatchers() 메서드와 regexMatchers() 메서드는 현재 Deprecated 상태다.

#### 스프링 시큐리에는 다음과 같은 세 유형의 선택기 메서드가 있다.
- MVC 선택기 - 경로에 MVC 식을 이용해 엔드포인트를 선택한다.
- 앤트 선택기 - 경로에 앤트 식을 이용해 엔드포인트를 선택한다.
- 정규식 선택기 - 경로에 정규식(regex)을 이용해 엔드포인트를 선택한다.

## 8.1 선택기 메서드로 엔드포인트 선택

#### mvcMatchers() 메서드
```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests(authorizationManagerRequestMatcherRegistry -> {
            authorizationManagerRequestMatcherRegistry
                    .mvcMatchers("/hello").hasRole("ADMIN")
                    .mvcMatchers("/ciao").hasRole("MANAGER");
        });

        return http.build();
    }
```

#### 나머지 엔드포인트에 인증되지 않은 접근 요청을 허용하도록 명시적으로 지정
```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests(authorizationManagerRequestMatcherRegistry -> {
            authorizationManagerRequestMatcherRegistry
                    .mvcMatchers("/hello").hasRole("ADMIN")
                    .mvcMatchers("/ciao").hasRole("MANAGER")
                    .anyRequest().permitAll();
        });

        return http.build();
    }
```

- 종종 경로만이 아닌 HTTP 방식도 지정해야 한다.

## 8.2 MVC 선택기로 권한을 부여할 요청 선택

- 스프링 시큐리티는 기본적으로 CSRF(사이트 간 요청 위조)에 대한 보호를 적용한다.
- http.csrf().disable() 메서드를 이용해 CSRF 보호를 비활성화할 수 있다.

## 8.3 앤트 선택기로 권한을 부여할 요청 선텍

#### 앤트 선택기에는 다음과 같은 세 메서드가 있다.
- antMatchers(HttpMethod method, String patterns) - 제한을 적용할 HTTP 방식과 경로를 참조할 앤트 패턴을 모두 지정할 수 있다.
- antMatchers(String patterns) - 경로만을 기준으로 권한 부여 제한을 적용할 때 더 쉽고 간단하게 이용할 수 있다.
- 모든 HTTP 방식에 자동으로 제한이 적용된다.
- antMatchers(HttpMethod method) - antMatchers(HttpMethod method, "/**")와 같은 의미이며 경로와 관계없이 특정 HTTP 방식을 지정할 수 있다.

> MVC 선택기를 이용하는 것이 좋다. 
> MVC 선택기를 이용하면 스프링의 경로 및 작업 매핑과 관련한 몇 가지 위험을 예방할 수 있다.

## 8.4 정규식 선택기로 권한을 부여할 요청 선택

#### 정규식 선택기에는 다음과 같은 두 메서드가 있다.
- regexMatchers(HttpMethod method, String regex) - 제한을 적용할 HTTP 방식과 경로를 참조할 정규식을 모두 지정한다. 같은 경로 그룹에 대해 HTTP 방식별로 다른 제한을 적용할 때 유용하다.
- regexMatchers(String regex) - 경로만을 기준으로 권한 부여 제한을 적용할 때 더 쉽고 간단하게 이용할 수 있다. 모든 HTTP 방식에 자동으로 제한이 적용된다.

## 요약

- 실제 시나리오에서는 요청마다 다른 권한 부여 규칙을 적용하는 경우가 많다.
- 경로와 HTTP 방식에 따라 권한 부여 규칙을 구성할 요청을 지정한다. 이를 위해 MVC, 앤트, 정규식 세 가지 선택기 메서드를 이용하는 방법을 배웠다.
- MVC와 앤트 선택기는 서로 비슷하며 일반적으로 이 중 하나를 선택해 권한 부여 제한을 적용할 요청을 지정할 수 있다.
- 요구 사항이 앤트나 MVC 식으로 해결하기에 너무 복잡할 때는 이보다 강력한 정규식으로 구현할 수 있다.
