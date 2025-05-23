# Chapter 12. 스프링 앱에서 데이터 소스 사용

## 12.1 데이터 소스

- 데이터 소스는 데이터베이스를 처리하는 서버(즉, 데이터베이스 관리 시스템 또는 DBMS)에 대한 커넥션을 관리하는 구성 요소다.
- 자바 앱에서 관계형 데이터베이스에 연결할 수 있는 언어 기능을 JDBC라고 한다.
- JDK는 특정한 기술(MySQL, 포스트그레스, 오라클)을 상세하게 구현하지 않는 대신 앱이 관계형 데이터베이스와 작업하는 데 필요한 객체에 대한 추상화만 제공한다.

> 데이터 소스는 앱의 데이터베이스 서버에 대한 커넥션을 관리하는 역할을 하는 객체다.

- 오늘날 가장 일반적으로 사용되는 것은 HikariCP(Hikari Connection Pool) 데이터 소스다. 스프링 부트의 관례 구성에서도 기본 데이터 소스로 사용하고 있다.

## 12.2 JdbcTemplate으로 영속성 데이터 작업

- double 또는 float 값으로 연산할 때는 더하기나 빼기 같은 간단한 산술 연산에서도 정밀도가 떨어질 수 있기 때문에 올바른 자료형 선택이 아닐 수도 있다.
- 가격 같은 민감한 정보를 작업할 때는 BigDecimal 타입을 사용해야 한다.

## 12.3 데이터 소스 구성을 사용자 정의

### 12.3.1 애플리케이션 프로퍼티 파일에서 데이터 소스 정의

> 프로덕션 단계의 앱에서 프로퍼티 파일에 시크릿을 저장하는 것은 좋은 행동이 아니다.
> 이런 개인 정보는 시크릿 볼트에 저장해야 한다.

## 12.4 요약

- 자바 애플리케이션은 JDK라는 앱이 관계형 데이터베이스 연결에 필요한 객체의 추상화를 제공한다. 앱은 항상 이런 추상화를 구현하는 런타임 의존성을 추가해야 한다. 이런 의존성을 JDBC 드라이버라 한다.
- 데이터 소스는 데이터베이스 서버에 대한 커넥션을 관리하는 객체다. 데이터 소스가 없다면 앱이 커넥션을 너무 자주 요청해서 성능에 영향을 미친다.
- 스프링 부트는 HikariCP라는 데이터 소스 구현을 기본적으로 구성한다. 이 HikariCP는 커넥션 풀로 앱이 데이터베이스에 대한 커넥션을 사용하는 방식을 최적화한다. 앱에 다른 데이터 소스 구현체가 더 적절하다면 다른 것을 사용할 수 있다.
- JdbcTemplate은 JDBC를 사용하여 관계형 데이터베이스에 액세스하려고 작성하는 코드를 간소화하는 스프링 도구다. JdbcTemplate 객체는 데이터베이스 서버에 접속하려고 데이터 소스에 의존한다.
- 테이블의 데이터를 변경하는 쿼리를 전송하려면 JdbcTemplate 객체의 update() 메서드를 사용한다. 데이터를 검색할 수 있는 SELECT 쿼리는 query() 메서드 중 하나를 사용한다.
- 스프링 부트 애플리케이션에서 사용되는 데이터 소스를 사용자 정의하려면 java.sql.Datasource 타입의 빈을 맞춤 구성한다. 스프링 컨텍스트에서 이 타입의 빈을 선언하면 스프링 부트는 기본 빈을 구성하지 않고 이 빈을 사용한다. 사용자 정의 JdbcTemplate 객체가 필요할 때도 동일한 방식을 사용한다. 특정 상황에서는 다양하게 최적화를 하려고 사용자 정의 구성 또는 구현이 필요할 때도 있다.
- 앱이 여러 데이터베이스에 연결하려면 고유한 JdbcTemplate 객체가 연결된 여러 데이터 소스 객체를 만들 수 있다. 애플리케이션 컨텍스트에서 동일한 타입의 객체를 구별하는 데 쓰는 @Qualifier 어노테이션을 사용하여 구별한다.