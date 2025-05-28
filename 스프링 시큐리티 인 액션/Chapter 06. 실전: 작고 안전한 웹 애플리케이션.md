# Chapter 06. 실전: 작고 안전한 웹 애플리케이션

## 6.1 프로젝트 요구 사항과 설정

### 의존성

- Spring web
- Spring Data JPA
- Spring Security
- Thymeleaf
- MySQL Driver

## 6.2 사용자 관리 구현

#### 두 암호 인코더의 빈 등록
```java
@Configuration
public class ProjectConfig {
    
    @Bean
    public BcryptPasswordEncoder bcryptPasswordEncoder() {
        return new BcryptPasswordEncoder(); 
    }
    
    @Bean
    public ScryptPasswordEncoder scryptPasswordEncoder() {
        return new ScryptPasswordEncoder(); 
    }
}
```

#### User 엔티티 클래스
```java
@Entity
public class User {
    
    @Id
    @GeneratedValue
    private Integer id;
    
    private String username;
    private String password;
    
    @Enumerated(EnumType.STRING)
    private EncryptionAlgorithm algorithm; // BCRYPT, SCRYPT
    
    @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
    private List<Authority> authorities;
    
    // getter, setter
}
```

#### Authority 엔티티 클래스
```java
@Entity
public class Authority {

    @Id
    @GeneratedValue
    private Integer id;
    
    private String name;
    
    @JoinColumn(name = "user")
    @ManyToOne
    private User user;
    
    // getter, setter
}
```

#### User 스프링 데이터 레포지토리
```java
public interface UserRepository extends JpaRepository<User, Integer> {
    
    Optional<User> findUserByUsername(String username);
}
```

#### UserDetails 인터페이스의 구현
```java
public class CustomUserDetails implements UserDetails {
    
    private final User user;
    
    public CustomUserDetails(User user) {
        this.user = user;
    }
    
    public final User getUser() {
        return user;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getAuthorities().stream()
                .map(a -> new SimpleGrantedAuthority(
                        a.getName()))
                .toList();
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

#### UserDetailsService 인터페이스의 구현
```java
@Service
public class JpaUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public CustomUserDetails loadUserByUsername(String username) {
        Supplier<UsernameNotFoundException> s = () -> new UsernameNotFoundException("problem during authentication!");
        
        User u = userRepository.findByUsername(username)
                .orElseThrow(s);
        
        return new CustomUserDetails(u);
    }
}
```

## 6.3 맞춤형 인증 논리 구현

#### AuthenticationProvider 구현
```java
@Service
public class AuthenticationProviderService implements AuthenticationProvider {

    @Autowired
    private final JpaUserDetailsService userDetailsService;

    @Autowired
    private final BCryptPasswordEncoder bCryptPasswordEncoder;

    @Autowired
    private final SCryptPasswordEncoder sCryptPasswordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();

        CustomUserDetails user= userDetailsService.loadByUsername(username);

        switch (user.getUser().getAlgorithm()) {
            case BCRYPT:
                return checkPassword(user, password, bCryptPasswordEncoder);
            case SCRYPT:
                return checkPassword(user, password, sCryptPasswordEncoder);
        }

        throw new BadCredentialsException("Bad credentials");
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(aClass);
    }
}
```

#### formLogin을 인증 메서드로 구성
- 현재 스프링 시큐리티 버전에 맞게 수정한 것입니다.
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
            .formLogin(httpSecurityFormLoginConfigurer -> {
        httpSecurityFormLoginConfigurer.defaultSuccessUrl("/main", true);
    });

    http.authorizeHttpRequests(authorizationManagerRequestMatcherRegistry -> {
        authorizationManagerRequestMatcherRegistry.anyRequest().authenticated();
    });

    return http.build();
}
```

## 요약

- 실제 애플리케이션에서는 같은 개념의 다른 구현이 필요한 종속성이 흔하게 사용된다. 이 예제에서는 스프링 시큐리티의 UserDetails와 JPA 구현의 User 엔티티가 이러한 사례에 해당하며, 이 때는 책임을 다른 클래스로 분리해 가독성을 높이는 것이 좋다.
- 대부분은 같은 기능을 여러 가지 다른 방법으로 구현할 수 있으며, 가장 단순한 해결책을 선택해야 한다. 코드를 이해하기 쉽게 만들면 오류와 보안 침해의 여지가 줄어든다.
