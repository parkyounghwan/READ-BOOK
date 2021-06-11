# 3\_데이터 접근\(JDBC, Tx\)

> p129-177

### 3.1.1. 데이터 소스 개요

데이터 소스는 애플리케이션이 데이터베이스에 접근하기 위한 추상화된 연결 방식, 즉 커넥션\(java.sql.Connection\)을 제공하는 역할을 한다.

* 애플리케이션 모듈이 제공

  서드파티가 제공하는 데이터 소스나 DriverManagerDataSource같이 스프링 프레임워크가 테스트 용도로 제공하는 데이터 소스를 빈으로 등록해서 사용하는 방식을 말한다.

  데이터베이스에 접속하기 위한 사용자 ID와 패스워드, 접속 대상 URL 같은 정보를 직접 관리하고 설정한다.

* 애플리케이션 서버가 제공

  JNDI를 통해 가져와서 사용하는 방식이다.

  데이터베이스에 접속하기 위한 각종 정보를 애플리케이션 서버에서 관리하기 때문에 애플리케이션으로부터 데이터베이스 관련 정보를 분리해낼 수 있을뿐더러 애플리케이션 서버가 제공하는 각종 관리 기능도 활용할 수 있다.

* 내장형 데이터베이스를 사용

  사전에 데이터베이스를 준비할 필요 없이 애플리케이션이 기동할 때 데이터 소스의 설정과 생성이 자동으로 이뤄진다.

  간편하게 사용할 수 있는 만큼 제약도 있기 때문에 미션 크리티컬한 엔터프라이즈 환경에서는 좀처럼 사용되지 않는다.

### 3.1.2. 데이터 소스 설정

메이븐의 pom.xml에 의존 관계를 설정한다.

### 3.2.1. 스프링 JDBC 개요

SQL의 실제 내용과는 상관없이 공통적이면서도 반복적으로 수행되는 JDBC 처리를 개발자가 직접 구현하는 대신 프레임워크가 대행하는 기능을 제공한다.

* 커넥션의 연결과 종료
* SQL 문의 실행
* SQL 문 실행 결과 행에 대한 반복 처리
* 예외 처리

위와 같은 공통적이고 반복적인 작업을 개발자가 직접 구현하지 않아도 된다. 개발자는 아래만 구현하면 된다.

* SQL 문 정의
* 파라미터 설정
* ResultSet에서 결과를 가져온 후, 각 레코드별로 필요한 처리

### 3.2.2. JdbcTemplate 클래스를 활용한 CRUD

`queryForObject` 하나의 결과 레코드 중에서 하나의 칼럼 값을 가져올 때 사용함. RowMapper와 함께 사용하면 하나의 레코드 정보를 객체에 매핑할 수 있음.

`queryForMap` 하나의 결과 레코드 정보를 Map 형태로 매핑할 수 있음.

`queryForList` 여러 개의 결과 레코드를 다룰 수 있음. List의 한 요소가 한 레코드에 해당. 한 레코드의 정보는 queryForObject나 queryForMap을 사용할 때와 같음.

`update` 데이터를 변경하는 SQL\(insert delete update\)을 실행할 때 사용함.

* 칼럼 값을 조회할 때 바인드 변수 사용

SQL문의 가변적인 부분에 '?' 와 같은 플레이스홀더를 두고 SQL을 실행하는 시점에 변수로 치환하는 방법이 있다. 이 변수를 바인드 변수라고 한다.

```java
public String findRoomNameById(String roomId) {
    String sql = "SELECT room_name FROM room WHERE room_id = ?";
    return jdbcTemplate.queryForObject(sql, String.class, roomId);
    // Object 배열도 가능
}
```

이는 바인딩할 파라미터의 개수가 많아질수록 가독성이 떨어진다.

* 네임드 파라미터 사용

```java
@Component
public String findRoomNameById(String roomId) {
    @Autowired
    NamedParameterJdbcTemplate ...;
    String sql = "SELECT room_name FROM room WHERE room_id = :roomId";
    Map<String, Object> params = new HashMap<String, Object>();
    params.put("roomId", roomId);
    return namedParameterJdbcTemplate.queryForObject(sql, String.class, roomId);
}
```

DAO 클래스를 정의하기 때문에 컴포넌트 스캔의 대상이 되도록 @Component 애너테이션을 붙여준다.

`:변수명` 형태로 플레이스홀더를 설정한다.

Map에 바인드 변수의 이름을 키로 사용하는 값을 넣는다.

* 조회 결과 레코드 값이 1건인 경우

사용자가 원하는 빈 형태로 가져오지 못하고 자바에서 기본적으로 제공하는 타입만 써야 한다. 그래서 조회 결과는 칼럼명을 키로 사용하는 Map 형태로 받게 되고, 값을 꺼낼 때는 적절한 타입으로 형변환한다.

* 조회 결과 레코드 값이 여러 건인 경우

한 레코드의 Map타입, 이 Map을 여러 개 담은 List 타입으로 반환한다.

* 조회 결과 레코드 값이 0건인 경우

`queryForList` 메서드는 빈 List를 반환하고 그 밖의 메서드에서는 EmptyResultDataAccessException예외를 던진다.

### 3.2.3. SQL 질의 결과를 POJO로 변환

보통은 해당 비즈니스에 맞는 데이터 타입을 POJO 형태로 만들어 쓰는 경향이 있기 때문에 반환값을 가공한다.

* RowMapper

ResultSet을 순차적으로 읽으면서 한 행씩 하나의 POJO객체로 변환한다. 다음 행으로 넘어가는 커서를 스프링이 제어해준다.

* ResultSetExtractor

ResultSet을 자유롭게 제어하면서 원하는 POJO형태로 매핑하고 싶을 때 사용한다. 다음 행으로 이동하는 next 메서드를 제공한다.

* RowCallbackHandler

반환값이 없다. 별도의 다른 처리를 하고 싶을 때 사용한다. 데이터를 파일 형태로 출력, 조회된 데이터 검증하는 용도.

## 3.3. 트랜잭션 관리

### 3.3.1. 트랜잭션 관리자

스프링 트랜잭션 인터페이스는 PlatformTransactionManager다. 이 인터페이스는 트랜잭션 처리에 필요한 API를 제공하며 개발자가 API를 호출하는 것으로 트랜잭션 조작을 수행할 수 있다. 다만 직접 호출하지 않고 스프링이 제공하는 편리한 API를 이용하는 사례가 더 많아지고 있다.

그럼 난 이건 넘어가고 애너테이션 부분을 봐야겠다.

### 3.3.2. 선언적 트랜잭션

@Transactional 애너테이션을 추가하면 대상 메서드의 시작 종료에 맞춰 트랜잭션을 시작, 커밋할 수 있다. 스프링의 기본 상태에서는 메서드 안의 처리에서 데이터 접근 예외와 같은 비검사 예외\(unchecked exception\)가 발생해서 메서드 안에 처리가 중단될 때 트랜잭션이 자동으로 롤백된다.

클래스와 메서드에 부여할 수 있다. 메서드에 부여하면 메서드 단위에서 개별적으로 트랜잭션 제어 방식을 설정할 수 있다. 클래스에 먼저 설정되었더라도 메서드에 지정된 설정이 우선된다.

### 3.3.3. 명시적 트랜잭션

커밋과 롤백과 같은 처리를 소스코으데 직접 기술하는 방법이다. 메서드 단위보다도 더 작은 단위로 트랜잭션을 제어하고 싶거나, 선언적 트랜잭션으로는 표현하기 어려운 섬세한 트랜잭션 제어가 필요할 때 이 방법을 사용한다.

`PlatformTransactionManager`와 `TransactionTemplate`을 사용하는 두 가지 방법을 제공한다.

### 3.3.4. 트랜잭션 격리 수준과 전파 방식

* 트랜잭션 격리 수준은 참조하는 데이터나 변경한 데이터를 다른 트랜잭션으로부터 어떻게 격리할 것인지를 결정한다. 스프링의 기본값인 DEFAULT에서 다른 수준으로 변경하고 싶은 경우에는 @Transactional의 `isolation` 속성, `TransactionDefinition`과 `TransactionTemplate`의 `setIsolationLevel` 메서드에서 지정할 수 있다.
* 트랜잭션 전파 방식은 트랜잭션 경계에서 트랜잭션에 참여하는 방법을 결정한다. 지원하는 방식에 따라 '새로운 트랜잭션을 시작하는 것', '이미 시작된 트랜잭션에 참여하는 것'과 같이 몇 가지 선택지가 준비돼 있다.

  트랜잭션 전파 방식을 의식해야 하는 경우는 트랜잭션 경계가 중첩될 때다 \(트랜잭션 관리 대상이 되는 메서드 안에서 또 다른 트랜잭션 관리 대상이 되는 메서드를 호출한 경우\). 여러 개의 트랜잭션 경계가 중첩되지 않았다면 트랜잭션을 순차적으로 제어만 하면 되기 때문에 굳이 전파 방식을 의식할 필요는 없다.

### 3.4.1. 데이터 접근 관련 예외

* `DataAccessException`을 부모 클래스로 하는 계층 구조

  데이터 접근과 관련된 모든 예외는 이 계층 구조를 토대로 데이터 접근 추상화를 구현한다.

  `DataAccessException`은 `RuntimeException`의 자식 클래스로 구현돼 있다. 이로써 `DataAccessException`은 비검사 예외가 된다. 비검사 예외는 굳이 처리할 필요가 없다면 예외 처리 코드를 생략해도 된다.

