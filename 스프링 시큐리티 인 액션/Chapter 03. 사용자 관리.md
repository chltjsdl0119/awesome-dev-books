# Chapter 03. 사용자 관리

- 모든 견고한 프레임워크는 계약을 이용해서 프레임워크의 구현과 이에 기반을 둔 애플리케이션을 분리한다.

## 3.1 스프링 시큐리티의 인증 구현

- UserDetailsService는 사용자 이름으로 사용자를 검색하는 역할만 한다. 이 작업은 프레임워크가 인증을 완료하는 데 반드시 필요한 유일한 작업이다.
- UserDetailsManager는 대부분의 애플리케이션에 필요한 사용자 추가, 수정, 삭제 작업을 추가한다.
- 이와 같은 두 계약 간의 분리는 인터페이스 분리 원칙의 훌륭한 예이다. 인터페이스를 분리하면 앱에 필요 없는 동작을 구현하도록 프레임워크에서 강제하지 않기 때문에 유연성이 향상된다.

## 3.2 사용자 기술하기

- 스프링 시큐리티에서 사용자 정의는 UserDetails 계약을 준수해야 한다.
- UserDetails 계약은 스프링 시큐리티가 이해하는 방식으로 사용자를 나타낸다.
- 애플리케이션에서 사용자를 기술하는 클래스는 프레임워크가 이해할 수 있도록 이 인터페이스를 구현해야 한다.

### 3.2.1 UserDetails 계약의 정의 이해하기

```java
public interface UserDetails extends Serializable {
    String getPassword();
    String getUsername();
    
    Collection<? extends GrantedAuthority> getAuthorities(); // 사용자에게 부여된 권한의 그룹을 반환한다.

    default boolean isAccountNonExpired() {
        return true;
    }

    default boolean isAccountNonLocked() {
        return true;
    }

    default boolean isCredentialsNonExpired() {
        return true;
    }

    default boolean isEnabled() {
        return true;
    }
}
```

- 아래의 다섯 메서드는 모두 사용자가 애플리케이션의 리소스에 접근할 수 있도록 권한을 부여하기 위한 것이다.

> 애플리케이션의 논리에서 이러한 사용자 제한을 구현하려면 isAccountNonExpired(), isAccountNonLocked(), isCredentialsNonExpired(), isEnabled() 메서드를 재정의해서 true를 반환하게 해야한다.

### 3.2.2 GrantedAuthority 계약 살펴보기

 - 권한은 사용자가 애플리케이션에서 수행할 수 있는 작업을 나타낸다.
 - GrantedAuthority 인터페이스 내의 getAuthority() 메서드는 권한을 String 값으로 반환한다. 람다식으로도 반환이 가능하다.

> 람다식으로 구현하기 전에 @FunctionalInterface 어노테이션을 지정하여 인터페이스가 함수형임을 지정하는 것이 좋다.
> 인터페이스가 함수형으로 표시되어 있지 않으면 인터페이스의 개발자가 향후 버전에 더 많은 추상 메서드를 추가할 권리가 있다는 의미일 수도 있기 때문이다.

3.2.3 최소한의 UserDetails 구현 작성

#### 정적 값을 반환하는 기본 구현. 현실적인 구현과 거리가 멀다.
```java
public class DummyUser implements UserDetails {

    @Override
    public String getUsername() {
        return "bill";
    }

    @Override
    public String getPassword() {
        return "12345";
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> "READ");
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}

```

#### 더 현실적인 UserDetails 인터페이스의 구현
```java
public class SimpleUser implements UserDetails {

    private final String username;
    private final String password;

    public SimpleUser(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    @Override
    public String getPassword() {
        return this.password;
    }
    
    // 생략된 코드
}

```

### 3.2.4 빌더를 이용해 UserDetails 형식의 인스턴스 만들기

```java
User.UserBuilder builder = User.withUsername("bill");

UserDetails u = User.withUsername("bill")
        .password("12345")
        .authorities("read", "write")
        .accountExpired(false)
        .disabled(true)
        .build();
```

3.2.5 사용자와 연관된 여러 책임 결합

#### 두 책임을 가진 User 클래스
```java
package com.springsecurity.dummy;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.List;

@Entity
public class User implements UserDetails {

    @Id
    private Long id;
    private String username;
    private String password;
    private String authority;

    // getter, setter

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of();
    }

    @Override
    public String getPassword() {
        return "";
    }

    @Override
    public String getUsername() {
        return "";
    }
    
    // 생략된 코드
}

```

- 애플리케이션에 두 책임이 필요한 것은 맞지만, 모두 한 클래스에 넣을 필요는 없다.

#### SecurityUser 클래스로 책임 분할
```java
public class SecurityUser implements UserDetails {

    private final User user;

    public SecurityUser(User user) {
        this.user = user;
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of();
    }
}
```

## 3.3 스프링 시큐리티가 사용자를 관리하는 방법 지정

### 3.3.1 UserDetailsService 계약의 이해

```java
public interface UserDetailsService {
    
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

- UserDetailsService 계약은 위와 같이 한 메서드만 포함한다.
- 이 메서드가 반환하는 사용자는 UserDetails 계약의 구현이다.

### 3.3.2 UserDetails 계약 구현

#### UserDetails 구현
```java
public class SimpleUser implements UserDetails {

    private final String username;
    private final String password;
    private final String authority;

    public SimpleUser(String username, String password, String authority) {
        this.username = username;
        this.password = password;
        this.authority = authority;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    @Override
    public String getPassword() {
        return this.password;
    }


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> authority);
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

#### UserDetailsService 구현
```java
public class InMemoryUserDetailsService implements UserDetailsService {

    private final List<UserDetails> users;

    public InMemoryUserDetailsService(List<UserDetails> users) {
        this.users = users;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return users.stream()
                .filter(u -> u.getUsername().equals(username))
                .findFirst()
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
    }
}
```

#### 빈 등록
```java
@Configuration
public class ProjectConfig {

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails u = new SimpleUser("user", "12345", "READ");
        List<UserDetails> users = List.of(u);
        return new InMemoryUserDetailsService(users);
    }
}
```

### 3.3.3 UserDetailsManager 계약 구현

- 2장에서 이용한 InMemoryUserDetailsManager 객체는 사실 UserDetailsManager 였다.

사용자 관리에 JdbcUserDetailsManager 이용

- 데이터베이스에 저장된 사용자를 관리하며 JDBC를 통해 데이터베이스에 직접 연결한다.

사용자 관리에 LdapUserDetailsManager 이용

- 이 부분은 현재 구현이 안됩니다.

## 요약

- UserDetails 인터페이스는 스프링 시큐리티에서 사용자를 기술하는 데 이용되는 계약이다.
- UserDetailsService 인터페이스는 애플리케이션이 사용자 세부 정보를 얻는 방법을 설명하기 위해 스프링 시큐리티의 인증 아키텍처에서 구현해야 하는 계약이다.
- UserDetailsManager 인터페이스는 UserDetails를 확장하고 사용자 생성, 변경, 삭제와 관련된 동작을 추가한다.
- 스프링 시큐리티는 UserDetailsManager 계약의 여러 구현을 제공한다. InMemoryUserDetailsManager, JdbcUserDetailsManager가 있다.
- JdbcUserDetailsManager는 JDBC를 직접 이요하므로 애플리케이션이 다른 프레임워크에 고정되지 않는다는 이점이 있다.
