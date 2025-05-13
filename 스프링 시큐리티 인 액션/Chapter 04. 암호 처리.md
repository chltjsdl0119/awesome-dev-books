# Chapter 04. 암호 처리

## 4.1 PasswordEncoder 계약의 이해

- 일반적으로 시스템은 암호를 일반 텍스트로 관리하지 않고 공격자가 암호를 읽고 훔치기 어렵게 하기 위한 일종의 변환 과정을 거친다.

### 4.1.1 PasswordEncoder 계약의 정의

- 모든 시스템은 어떤 방식으로든 인코딩된 암호를 저장하며 아무도 암호를 읽을 수 없게 해시를 저장해야 한다.

#### PasswordEncoder 인터페이스
```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);

    boolean matches(CharSequence rawPassword, String encodedPassword);

    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

- upgradeEncoding 메서드는 계약에서 기본값 false를 반환한다. true를 반환하도록 메서드를 재정의하면 인코딩된 암호를 보안 향상을 위해 다시 인코딩한다.
- 인코딩된 암호를 다시 인코딩하면 결과에서 원래 암호를 알아내기가 더 어려울 수 있지만 모호하다.

### 4.1.2 PasswordEncoder 계약의 구현

- matches() 메서드와 encode() 메서드는 기능 면에서 항상 일치해야 한다.

#### 가장 단순한 PasswordEncoder 구현
```java
public class CustomPasswordEncoder implements PasswordEncoder {
    @Override
    public String encode(CharSequence rawPassword) {
        return rawPassword.toString();
    }

    @Override
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        return rawPassword.equals(encodedPassword);
    }
}
```

#### SHA-512를 이용하는 PasswordEncoder 구현
```java
public class CustomPasswordEncoder implements PasswordEncoder {
    @Override
    public String encode(CharSequence rawPassword) {
        return hashWithSHA512(rawPassword.toString());
    }

    @Override
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        String hashedPassword = encode(rawPassword);
        return encodedPassword.equals(hashedPassword);
    }

    private String hashWithSHA512(String input) {
        StringBuilder result = new StringBuilder();

        try {
            MessageDigest md = MessageDigest.getInstance("SHA-512");
            byte[] digested = md.digest(input.getBytes());
            for (int i = 0; i < digested.length; i++) {
                result.append(Integer.toHexString(0xFF & digested[i]));
            }
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("Bad Algorithm");
        }

        return result.toString();
    }
}
```

### 4.1.3 PasswordEncoder의 제공된 구현 선택

- 스프링 시큐리티에 이미 몇 가지 유용한 구현이 있다.

> - NoOpPasswordEncoder: 실제 시나리오에서는 절대 쓰지 말아야 한다.
> - StandardPasswordEncoder: SHA-256을 이용한다. 이제는 구식이다.
> - Pdkdf2PasswordEncoder: PBKDF2를 이용한다.
> - BCryptPasswordEncoder: bcrypt 강력 해싱 함수로 인코딩한다.
> - SCryptPasswordEncoder: scrypt 해싱 함수로 인코딩한다.

- 해싱 알고리즘에 관한 자세한 내용은 <<Real-World Cryptography>> 참고.

#### BCryptPasswordEncoder 사용법
```java
PasswordEncoder p = new BCryptPasswordEncoder();
PasswordEncoder p = new BCryptPasswordEncoder(4);

SecureRandom s = SecureRandom.getInstanceString();
PasswordEncoder p = new BCryptPasswordEncoder(4, s);
```

- 강력 해싱 함수, bcrypt 함수를 사용하는 BCryptPasswordEncoder를 사용하는 것이 좋다.

### 4.1.4 DelegatingPasswordEncoder를 이용한 여러 인코딩 전략

- 현재 사용되는 알고리즘에서 취약성이 발견되어 신규 등록 사용자의 자격 증명을 변경하고 싶지만 기존 자격 증명은 변경하기가 쉽지 않다. 이 때는 여러 종류의 해시를 지원해야 하는데, DelegatingPasswordEncoder가 좋은 선택이다.
- DelegatingPasswordEncoder는 인코딩 알고리즘을 구현하는 대신 같은 계약의 다른 구현 인스턴스에 작업을 위임한다. 암호의 접두사를 기준으로 올바른 PasswordEncoder 구현에 작업을 위임한다.

#### DelegatingPasswordEncoder 인스턴스 만들기
```java
    @Bean
    public PasswordEncoder passwordEncoder() {
        Map<String, PasswordEncoder> encoders = new HashMap<>();

        encoders.put("noop", NoOpPasswordEncoder.getInstance());
        encoders.put("bcrypt", new BCryptPasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());

        return new DelegatingPasswordEncoder("bcrypt", encoders);
    }
```

- PasswordEncoderFactories 클래스에는 bcrypt가 기본 인코더인 DelegatingPasswordEncoder의 구현을 반환하는 정적 메서드 createDelegatingPasswordEncoder()가 있다.

#### PasswordEncoderFactories.createDelegatingPasswordEncoder() 메서드 사용
```java
PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

#### 지금까지 언급된 계약
| 인터페이스명             | 설명                                                                                     |
|--------------------|----------------------------------------------------------------------------------------|
| UserDetails        | 스프링 시큐리티가 관리하는 사용자를 나타낸다.                                                              |
| GrantedAuthority   | 애플리케이션의 목적 내에서 사용자에게 허용되는 작업을 정의한다.(읽기, 쓰기, 삭제 등)                                      |
| UserDetailsService | 사용자 이름으로 사용자 세부 정보를 검색한다.                                                              |
| UserDetailsManager | UserDetailsService보다 더 구체적인 계약이다. 사용자 이름으로 사용자를 검색하는 것외에도 사용자 컬렉션이나 특정 사용자를 변경할 수도 있다. |
| PasswordEncoder    | 암호를 암호화 또는 해시하는 방법과 주어진 인코딩된 문자열을 일반 텍스트 암호와 비교하는 방법을 지정한다.                            |

## 4.2 스프링 시큐리티 암호화 모듈에 관한 추가 정보

- 개발자의 개발 노력을 지원하기 위해 프로젝트의 종속성을 줄일 수 있는 **키 생성기**, 암호기 등의 자체 솔루션을 제공한다.

### 4.2.1 키 생성기 이용

- 키 생성기는 특정한 종류의 키를 생성하는 객체로서 일반적으로 암호화나 해싱 알고리즘에 필요하다.

#### StringKeyGenerator 계약의 정의
```java
public interface StringKeyGenerator {
    String generatorKey();
}
```

#### StringKeyGenerator 인스턴스를 얻고 솔트 값을 가져오는 방법
```java
StringKeyGenerator keyGenerator = KeyGenerators.string();
String salt = KeyGenerators.generateKey();
```

#### ByteKeyGenerator 계약의 정의
```java
public interface ByteKeyGenerator {
    int getKeyLength();
    byte[] generateKey();
}
```

#### 8바이트 길이의 키를 생성하는 방법
```java
ByteKeyGenerator keyGenerator = KeyGenerators.secureRandom();
byte [] key = KeyGenerators.generateKey();
int keyLength = KeyGenerators.getKeyLength();
```

### 4.2.2 암호화와 복호화 작업에 암호기 이용

- 암호기는 암호화 알고리즘을 구현하는 객체다.
- 암호기는 암호화와 복호화 작업을 지원하며 SSCM에는 이를 위해 ByteEncryptor 및 TextEncryptor라는 두 유형의 암호기가 정의되어 있다.

#### TextEncryptor 인터페이스
```java
public interface TextEncryptor {
    String encrypt(String text);
    String decrypt(String encryptedText);
}
```

#### ByteEncryptor 인터페이스
```java
public interface ByteEncryptor {
    byte[] encrypt(byte[] byteArray);
    byte[] decrypt(byte[] encryptedArray);
}
```

#### Encryptors로 ByteEncryptor 사용하기
```java
String salt = KeyGenerators.string().generateKey();
String password = "secret";
String valueToEncrypt = "HELLO";

ByteEncryptor e = Encryptors.standard(password, salt); // 256바이트 AES 암호화를 이용, CBC 이용

ByteEncryptor e = Encryptors.stronger(password, salt); // 더 강력한 암호기 인스턴스, GCM 이용

byte[] encrypted = e.encrypt(valueToEncrypt.getBytes(0));
byte[] decrypted = e.decrypt(encrypted);
```

#### Encryptors로 TextEncryptor 사용하기
```java
String salt = KeyGenerators.string().generateKey();
String password = "secret";
String valueToEncrypt = "HELLO";

TextEncryptor e = Encryptors.text(password, salt);
String encrypted = e.encrypt(valueToEncrypt);
String decrypted = e.decrypt(encrypted);
```

#### Encryptors.queryableText()로 같은 입력에 대해 같은 출력값 내기
```java
String salt = KeyGenerators.string().generateKey();
String password = "secret";
String valueToEncrypt = "HELLO";


TextEncryptor e = Encryptors.queryableText(password, salt);

// 아래 encrypted1과 encrypted2는 같은 값을 가지고 있다.
String encrypted1 = e.encrypt(valueToEncrypt);
String encrypted2 = e.encrypt(valueToEncrypt);
```

- 항상 다른 출력을 반환하는 방식을 원하지 않을 땐 위 방식을 채택한다.

## 요약

- PasswordEncoder는 인증 논리에서 암호를 처리하는 가장 중요한 책임을 담당한다.
- 스프링 시큐리티에서는 해싱 알고리즘에 여러 대안을 제공하므로 필요한 구현을 선택하면 된다.
- 스프링 시큐리티 암호화 모듈(SSCM)에는 키 생성기와 암호기를 구현하는 여러 대안이 있다.
- 키 생성기는 암호화 알고리즘에 이용되는 키를 생성하도록 도와주는 유틸리티 객체다.
- 암호기는 데이터 암호화와 복호화를 수행하도록 도와주는 유틸리티 객체다.
