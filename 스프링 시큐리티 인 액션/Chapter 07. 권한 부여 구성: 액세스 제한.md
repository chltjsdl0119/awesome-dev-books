# Chapter 07. 권한 부여 구성: 액세스 제한

- 특정 기능이나 데이터에 접근하려면 누구든지 올바른 절차를 거쳐야 한다.

## 7.1 권한과 역할에 따라 접근 제한

### 7.1.1 사용자 권한을 기준으로 모든 엔드포인트에 접근 제한

#### WRITE 권한이 있는 사용자만 접근 허용
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    
    http.authorizeHttpRequests(authorizationManagerRequestMatcherRegistry -> {
        authorizationManagerRequestMatcherRegistry.anyRequest().hasAuthority("WRITE"); // 사용자가 엔드포인트에 접근하기 위한 조건 지정
    });

    return http.build();
}
```

#### hasAnyAuthority() 메서드로 여러 권한 허용
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    
    http.authorizeHttpRequests(authorizationManagerRequestMatcherRegistry -> {
        authorizationManagerRequestMatcherRegistry.anyRequest().hasAnyAuthority("WRITE", "READ");
    });

    return http.build();
}
```

### 7.1.2 사용자 역할을 기준으로 모든 엔드포인트에 대한 접근을 제한

- 스프링 시큐리티는 권한을 제한이 적용되는 결이 고운(fine grained) 이용 권리라고 이해한다.
- 역할을 정의할 때 역할 이름은 ROLE_ 접두사로 시작해야 한다.

#### 관리자의 요청만 수락하도록 앱 구성
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(authorizationManagerRequestMatcherRegistry -> {
        authorizationManagerRequestMatcherRegistry.anyRequest().hasRole("ADMIN"); // 역할을 선언할 때만 ROLE_ 접두사를 쓴다.
    });

    return http.build();
}
```

#### roles() 메서드로 역할 허용
```java
@Bean
public UserDetailsService userDetailsService() {
    var manager = new InMemoryUserDetailsManager();

    var user1 = User.withUsername("john")
            .password("1234")
            .roles("ADMIN")
            .build();

    var user2 = User.withUsername("jane")
            .password("1234")
            .roles("MANAGER")
            .build();

    manager.createUser(user1);
    manager.createUser(user2);

    return manager;
}
```

### 7.1.3 모든 엔드포인트에 대한 접근 제한

- denyAll() 메서드를 사용하여 모든 엔드포인트에 대한 접근을 제한할 수 있다.

## 요약

- 권한 부여는 애플리케이션이 인증된 요청을 허가할지 결정하는 프로세스다. 권한 부여는 항상 인증 이후에 수행된다.
- 애플리케이션이 인증된 사용자의 권한과 역할에 따라 권한을 부여하는 방법을 구성할 수 있다.
- 애플리케이션에서 인증되지 않은 사용자가 특정 요청을 수행할 수 있게 지정할 수 있다.
- denyAll() 메서드로 앱이 모든 요청을 거부하고 permitALl() 메서드로 모든 요청을 수락하게 할 수 있다.
