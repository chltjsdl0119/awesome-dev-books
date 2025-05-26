# Chapter 05. 인증 구현

- 인증 논리를 담당하는 것은 AuthenticationProvider 계층이다.
- AuthenticationProvider 계층에서 요청을 허용할지 결정하는 조건과 명령을 반견할 수 있다.

## 5.1 AuthenticationProvider의 이해

> 어떠한 시나리오가 필요하더라도 구현할 수 있게 해주는 것이 프레임워크의 목적이다.

### 5.1.1 인증 프로세스 중 요청 나타내기

- 애플리케이션에 접근을 요청하는 사용자를 **주체(Principal)** 라고 한다.
- 자바 시큐리티 API의 Principal 인터페이스를 확장한 것이 Authentication 인터페이스이다.

#### Authentication 인터페이스
```java
public interface Authentication extends Principal, Serializable {
    
	Collection<? extends GrantedAuthority> getAuthorities();
    
	Object getCredentials();

	Object getDetails();

	Object getPrincipal();

	boolean isAuthenticated();
    
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;

}
```

> 위 계약에서 알아야 할 메서드
> - isAuthenticated() - 인증 프로세스가 끝났으면 true, 아직 진행 중이면 false를 반환한다.
> - getCredentials() - 인증 프로세스에 이용된 암호나 비밀을 반환한다.
> - getAuthorities() - 인증된 요청에 허가된 권한의 컬렉션을 반환한다.

### 5.1.2 맞춤형 인증 논리 구현

- AuthenticationProvider 인터페이스의 기본 구현은 시스템의 사용자를 찾는 책임을 UserDetailsService에 위임하고 PasswordEncoder로 인증 프로세스에서 암호를 관리한다.

#### AuthenticationProvider 인터페이스
```java
public interface AuthenticationProvider {
    
	Authentication authenticate(Authentication authentication) throws AuthenticationException;

	boolean supports(Class<?> authentication);

}
```

- AuthenticationProvider 책임은 Authentication 계약과 강하게 결합되어 있다.
- supports() 메서드는 Authentication 객체로 제공된 형식을 지원하면 true를 반환한다.

> authenticate() 메서드 구현하는 방법
> - 인증이 실패하면 메서드는 AuthenticationException을 던져야 한다.
> - 메서드가 현재 AuthenticationProvider 구현에서 지원되지 않는 인증 객체를 받으면 null을 반환해야 한다.
> - 메서드는 완전히 인증된 객체를 나타내는 Authentication 인스턴스를 반환해야 한다.

### 5.1.3 맞춤형 인증 논리 적용

#### AuthenticationProvider 구현
```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();

        UserDetails u = userDetailsService.loadUserByUsername(username);

        if (passwordEncoder.matches(password, u.getPassword())) {
            return new UsernamePasswordAuthenticationToken(username, password, u.getAuthorities());
        } else {
            throw new BadCredentialsException("something went wrong!");
        }
    }

    @Override
    public boolean supports(Class<?> authenticationType) {
        return authenticationType.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

## 교훈

- 프레임워크, 특히 애플리케이션에 널리 이용되는 프레임워크는 수많은 똑똑한 개발자의 참여로 개발된다.
- 프레임워크를 이용하기로 했다면 최소한 프레임워크의 기본은 이해해야 한다.
- 프레임워크를 이용하기로 결정했다면 최대한 프레임워크의 의도된 용도에 맞게 이용한다.
- 프레임워크가 제공하는 것보다 맞춤형 코드를 작성하는 일이 많다고 느낀다면 이런 일이 왜 생기는지 의문을 가져야 한다.
- 프레임워크는 충분히 테스트되고 취약성을 포함하는 변경이 적다.
- 프레임워크는 추상을 잘 활용하므로 유지 관리하기 편한 애플리케이션을 개발하기도 좋다.

## 5.2 SecurityContext 이용

- 인증 프로세스를 성공적으로 완료한 후 요청이 유지되는 동안 Authentication 인스턴스를 SecurityContext에 저장한다.

#### SecurityContext 인터페이스
```java
public interface SecurityContext extends Serializable {

	Authentication getAuthentication();
    
	void setAuthentication(Authentication authentication);
}
```

#### SecurityContext 자체는 어떻게 관리될까?

- 스프링 시큐리티는 관리자 역할을 하는 객체로 SecurityContext를 관리하는 세 가지 전략을 제공한다.

1. MODE_THREADLOCAL - 각 스레드가 SecurityContext에 각자의 세부 정보를 저장할 수 있게 해준다. 요청당 스레드 방식의 웹 애플리케이션에서는 각 요청이 개별 스레드를 가지므로 이는 일반적인 접근이다.
2. MODE_INHERITABLETHREADLOCAL - MODE_THREADLOCAL과 비슷하지만 비동기 메서드의 경우 SecurityContext를 다음 스레드로 복사하도록 스프링 시큐리티에 지시한다.
3. MODE_GLOBAL - 애플리케이션의 모든 스레드가 같은 SecurityContext 인스턴스를 보게 한다.

- 나중에 복습 때 정리해볼 예정입니다.
### 5.2.1 보안 컨텍스트를 위한 보유 전략 이용

### 5.2.2 비동기 호출을 위한 보유 전략 이용

### 5.2.3 독립형 애플리케이션을 위한 보유 전략 이용

## 5.3 HTTP Basic 인증과 양식 기반 로그인 인증 이해하기

### 5.3.1 HTTP Basic 이용 및 구성

## 요약

- AuthenticationProvider 구성 요소를 이용하면 맞춤형 인증 논리를 구현할 수 있다.
- 맞춤형 인증 논리를 구현할 때는 책임을 분리하는 것이 좋다. AuthenticationProvider는 사용자 관리는 UserDetailsService에 위임하고, 암호 검증 책임은 PasswordEncoder에 위임한다.
- SecurityContext는 인증이 성공한 후 인증된 엔티티에 대한 세부 정보를 유지한다.
- 보안 컨텍스트를 관리하는 데는 MODE_THREADLOCAL, MODE_INHERITABLETHREADLOCAL_ MODE_GLOBAL의 세 전략을 이용할 수 있으며, 선택한 전략에 따라 다른 스레드에서 보안 컨텍스트 세부 정보에 접근하는 방법이 달라진다.
- 공유 스레드 로컬 전략을 사용할 때는 스프링이 관리하는 스레드에만 전략이 적용된다는 것을 기억하자. 프레임워크는 자신이 관리하지 않는 스레드에는 보안 컨텍스트를 복사하지 않는다.
- 스프링 시큐리티는 코드에서 생성했지만 프레임워크가 인식한 스레드를 관리할 수 있는 우수한 유틸리티 클래스를 제공한다. 코드에서 생성한 스레드의 SecurityContext를 관리하기 위해 다음 클래스를 이용할 수 있다.
  - DelegatingSecurityContextRunnable
  - DelegatingSecurityContextCallable
  - DelegatingSecurityContextExecutor
- 스프링 시큐리티는 양식 기반 로그인 인증 메서드인 formLogin()으로 로그인 양식과 로그아웃하는 옵션을 자동으로 구성한다. 작은 웹 애플리케이션의 경욱 직관적으로 이용할 수 있다.
- formLogin 인증 메서드는 세부적으로 맞춤 구성이 가능하며 HTTP Basic 방식과 함께 이용해 두 인증 유형을 모두 지원할 수도 있다.
