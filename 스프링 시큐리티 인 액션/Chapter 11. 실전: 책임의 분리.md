# Chapter 11. 실전: 책임의 분리

## 11.1 예제의 시나리오와 요구 사항

## 11.2 토큰의 구현과 이용

- 웹 애플리케이션에서는 엔드포인트가 리소스를 나타낸다.

### 11.2.1 토큰이란?

- 토큰은 이론상 일종의 출입 카드다.

> 토큰의 장점
> - 토큰을 이용하면 요청할 때마다 자격 증명을 공유할 필요가 없다.
>   - 자격 증명을 더 자주 노출할수록 누군가 이를 가로챌 기회도 많아진다.
> - 토큰의 수명을 짧게 지정할 수 있다.
> - 자격 증명을 무효로 하지 않고 토큰을 무효로 할 수 있다.
> - 클라이언트가 요청할 때 보내야 하는 사용자 권환과 같은 세부 정보를 토큰에 저장할 수도 있다.
> - 토큰을 이용하면 인증 책임을 시스템의 다른 구성 요소에 위임할 수 있다.

### 11.2.2 JSON 웹 토큰이란?

- JWT는 세 부분으로 구성되고 각 부분은 마침표(.)로 구분된다.
- 처음 두 부분은 헤더와 본문이다. 헤더와 본문은 JSON으로 형식이 지정되고 Base64로 인코딩된다.


- 헤더에는 토큰과 관련된 메타데이터를 저장한다.
- 토큰은 가급적 짧게 유지하고 본문에 너무 많은 데이터를 추가하지 않는 것이 좋다.

> 알아야할 사항
> - 토큰이 너무 길면 요청 속도가 느려진다.
> - 토큰에 서명하는 경우 토큰이 길수록 암호화 알고리즘이 서명하는 시간이 길어진다.

- 토큰의 마지막 부분은 디지털 서명이며 이 부분은 생략할 수 있다.
- JWT는 토큰의 한 구현이다.

## 11.3 인증 서버 구현

#### 엔티티 클래스
```java
@Entity
@Getter
@Setter
public class User {

    @Id
    private String username;
    private String password;
}

@Entity
@Getter
@Setter
public class Otp {

    @Id
    private String username;
    private String code;
}

```

#### UserService
```java
@Service
@Transactional
@RequiredArgsConstructor
public class UserService {

    private final PasswordEncoder passwordEncoder;
    private final UserRepository userRepository;
    private final OtpRepository otpRepository;

    public void addUser(User user) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        userRepository.save(user);
    }

    public void auth(User user) {
        User findUser = userRepository.findUserByUsername(user.getUsername()).orElseThrow(() -> new BadCredentialsException("Bad Credential."));

        if (passwordEncoder.matches(user.getPassword(), findUser.getPassword())) {
            renewOtp(findUser);
        } else {
            throw new BadCredentialsException("Bad Credential.");
        }
    }

    public boolean check(Otp otpToValidate) {
        Optional<Otp> userOtp = otpRepository.findOtpByUsername(otpToValidate.getUsername());

        if (userOtp.isPresent()) {
            Otp otp = userOtp.get();

            if (otpToValidate.getCode().equals(otp.getCode())) {
                return true;
            }
        }

        return false;
    }

    private void renewOtp(User u) {
        String code = GenerateCodeUtil.generateCode();

        Optional<Otp> userOtp = otpRepository.findOtpByUsername(u.getUsername());

        if (userOtp.isPresent()) {
            Otp otp = userOtp.get();
            otp.setCode(code);
        } else {
            Otp otp = new Otp();
            otp.setUsername(u.getUsername());
            otp.setCode(code);
            otpRepository.save(otp);
        }
    }
}
```

#### GenerateCodeUtil
```java
public final class GenerateCodeUtil {

    private GenerateCodeUtil() {
    }

    public static String generateCode() {
        String code;

        try {
            SecureRandom random = SecureRandom.getInstanceStrong();

            int c = random.nextInt(9000) + 1000;

            code = String.valueOf(c);

        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }

        return code;
    }
}
```

#### AuthController
```java
@RestController
@RequiredArgsConstructor
public class AuthController {

    private final UserService userService;

    @PostMapping("/user/add")
    public void addUser(@RequestBody User user) {
        userService.addUser(user);
    }

    @PostMapping("/user/auth")
    public void auth(@RequestBody User user) {
        userService.auth(user);
    }

    @PostMapping("/otp/check")
    public void check(@RequestBody Otp otp, HttpServletResponse response) {
        if (userService.check(otp)) {
            response.setStatus(HttpServletResponse.SC_OK);
        } else {
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        }
    }
}
```

#### 구성 클래스
```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth
                        .anyRequest().permitAll())
        ;

        return http.build();
    }
}
```

## 11.4 비즈니스 논리 서버 구현

### 11.4.1 Authentication 객체 구현

#### UsernamePasswordAuthentication 클래스
```java
// 매개 변수가 2개인 생성자를 호출하면 인증 인스턴스가 인증되지 않은 상태로 유지되지만 매개 변수가 3개인 생성자를 호출하면 Authentication 객체가 인증된다.
public class UsernamePasswordAuthentication extends UsernamePasswordAuthenticationToken {

    public UsernamePasswordAuthentication(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
        super(principal, credentials, authorities);
    }

    public UsernamePasswordAuthentication(Object principal, Object credentials) {
        super(principal, credentials);
    }
}
```

#### OtpAuthentication 클래스
```java
public class OtpAuthentication extends UsernamePasswordAuthenticationToken {

    public OtpAuthentication(Object principal, Object credentials) {
        super(principal, credentials);
    }

    public OtpAuthentication(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
        super(principal, credentials, authorities);
    }
}
```

### 11.4.2 인증 서버에 대한 프록시 구현

#### User 모델 클래스
```java
@Getter
@Setter
public class User {

    private String username;
    private String password;
    private String code;
}
```

#### 구성 클래스
```java
@Configuration
public class ProjectConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

#### AuthenticationServerProxy 클래스
```java
@Component
@RequiredArgsConstructor
public class AuthenticationServerProxy {

    private final RestTemplate restTemplate;

    @Value("${auth.server.base.url}")
    private String baseUrl;

    public void sendAuth(String username, String password) {
        String url = baseUrl + "/user/auth";

        var body = new User();
        body.setUsername(username);
        body.setPassword(password);

        var request = new HttpEntity<>(body);

        restTemplate.postForEntity(url, request, Void.class);
    }

    public boolean sendOTP(String username, String code) {
        String url = baseUrl + "/otp/check";

        var body = new User();
        body.setUsername(username);
        body.setCode(code);

        var request = new HttpEntity<>(body);
        var response = restTemplate.postForEntity(url, request, Void.class);

        return response.getStatusCode().equals(HttpStatus.OK);
    }
}
```

### 11.4.3 AuthenticationProvider 인터페이스 구현

#### UsernamePasswordAuthenticationProvider 클래스
```java
@Component
@RequiredArgsConstructor
public class UsernamePasswordAuthenticationProvider implements AuthenticationProvider {

    private final AuthenticationServerProxy proxy;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = String.valueOf(authentication.getCredentials());

        proxy.sendAuth(username, password);

        return new UsernamePasswordAuthenticationToken(username, password);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthentication.class.isAssignableFrom(authentication);
    }
}
```

#### OtpAuthenticationProvider 클래스
```java
@Component
@RequiredArgsConstructor
public class OtpAuthenticationProvider implements AuthenticationProvider {

    private final AuthenticationServerProxy proxy;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String code = String.valueOf(authentication.getCredentials());

        boolean result = proxy.sendOTP(username, code);

        if (result) {
            return new OtpAuthentication(username, code);
        } else {
            throw new BadCredentialsException("Bad Credential.");
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return OtpAuthentication.class.isAssignableFrom(authentication);
    }
}
```

### 11.4.4 필터 구현

#### InitialAuthenticationFilter 클래스
```java
@Component
public class InitialAuthenticationFilter extends OncePerRequestFilter {

    private final AuthenticationManager authenticationManager;

    @Value("${jwt.signing.key}")
    private String signingKey;

    public InitialAuthenticationFilter(@Lazy AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String username = request.getHeader("username");
        String password = request.getHeader("password");
        String code = request.getHeader("code");

        if (code == null) {
            Authentication a = new UsernamePasswordAuthentication(username, password);
            authenticationManager.authenticate(a);
        } else {
            Authentication a = new OtpAuthentication(username, code);

            a = authenticationManager.authenticate(a);

            SecretKey key = Keys.hmacShaKeyFor(signingKey.getBytes(StandardCharsets.UTF_8));

            String jwt = Jwts.builder()
                    .setClaims(Map.of("username", username))
                    .signWith(key)
                    .compact();

            response.setHeader("Authorization", jwt);
        }
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
        return !request.getServletPath().equals("/login");
    }
}
```

#### JwtAuthenticationFilter 클래스
```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Value("${jwt.signing.key}")
    private String signingKey;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String jwt = request.getHeader("Authorization");

        SecretKey key = Keys.hmacShaKeyFor(signingKey.getBytes(StandardCharsets.UTF_8));

        Claims claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(jwt)
                .getBody();

        String username = String.valueOf(claims.get("username"));

        GrantedAuthority a = new SimpleGrantedAuthority("user");
        var auth = new UsernamePasswordAuthentication(username, null, List.of(a));

        SecurityContextHolder.getContext()
                .setAuthentication(auth);

        filterChain.doFilter(request, response);
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
        return request.getServletPath().equals("/login");
    }
}
```

### 11.4.5 보안 구성 작성

#### SecurityConfig 클래스
```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Autowired
    private InitialAuthenticationFilter initialAuthenticationFilter;

    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    @Autowired
    private OtpAuthenticationProvider otpAuthenticationProvider;

    @Autowired
    private UsernamePasswordAuthenticationProvider usernamePasswordAuthenticationProvider;

    @Bean
    public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder =
                http.getSharedObject(AuthenticationManagerBuilder.class);
        authenticationManagerBuilder.authenticationProvider(otpAuthenticationProvider);
        authenticationManagerBuilder.authenticationProvider(usernamePasswordAuthenticationProvider);
        return authenticationManagerBuilder.build();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(AbstractHttpConfigurer::disable)
                .addFilterAt(initialAuthenticationFilter, BasicAuthenticationFilter.class)
                .addFilterAfter(jwtAuthenticationFilter, BasicAuthenticationFilter.class)
                .authorizeHttpRequests(auth -> auth
                        .anyRequest().authenticated());

        return http.build();
    }
}
```

### 11.4.6 전체 시스템 테스트

## 요약

- 맞춤형 인증과 권한 부여를 구현할 때는 항상 스프링 시큐리티에 있는 계약을 이용한다. 이러한 계약에는 AuthenticationProvider, AuthenticationManager, UserDetailsService 등이 있다.
- 토큰은 사용자를 위한 식별자이며 생성한 후 서버가 인식할 수 있으면 어떤 구현이든 이용할 수 있다.
- JWT는 서명하거나 완전히 암호화할 수 있다. 서명된 JWT를 JWS라고 하며 세부 정보가 완전히 암호화된 토큰을 JWE라고 한다.
- JWT에는 너무 많은 세부 정보를 저장하지 않는 것이 좋다. 토큰이 서명되거나 암호화되면 토큰이 길수록 이를 서명하거나 암호화하는 데 시간이 많이 걸린다. 또한 토큰은 HTTP 요청에 헤더로 보낸다는 것을 기억하자. 토큰이 길면 각 요청에 추가되는 데이터가 증가하고 애플리케이션의 성능이 영향을 크게 받는다.
- 애플리케이션을 유지 관리하고 확장하기 편하도록 책임을 분리하는 것이 좋다.
- 다단계 인증(MFA)은 사용자가 리소스에 접근할 때 여러 번 다른 방법으로 인증하도록 요청하는 인증 전략이다.
- 한 가지 문제에 대한 좋은 해결책이 두 개 이상인 경우가 많다. 항상 가능한 모든 해결책을 고려하고 시간이 허용된다면 모든 옵션의 개념 증명을 구현해 시나리오에 가장 적합한 옵션을 찾는 것이 좋다.
