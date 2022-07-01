# Test
> 저자는 스링프링이 제공하는 가장 큰 가치는 객체지향과 테스트라고 했다. 
- [Test](#test)
  - [UserDaoTest 다시보기](#userdaotest-다시보기)
    - [UserDaoTest](#userdaotest)
      - [웹을 통한 DAO 테스트 방법의 문제점](#웹을-통한-dao-테스트-방법의-문제점)
      - [작은 단위 테스트](#작은-단위-테스트)
      - [자동 수행 테스트 코드](#자동-수행-테스트-코드)
      - [지속적인 개선과 점진적인 개발을 위한 테스트](#지속적인-개선과-점진적인-개발을-위한-테스트)
    - [UserDaoTest의 문제점](#userdaotest의-문제점)
  - [UserDaoTest 개선](#userdaotest-개선)
    - [테스트 검증의 자동화](#테스트-검증의-자동화)
    - [테스트의 효율적인 수행과 결과 관리](#테스트의-효율적인-수행과-결과-관리)
      - [JUnit 테스트로 전환](#junit-테스트로-전환)
      - [테스트 메소드 전환](#테스트-메소드-전환)
        - [Junit 프레임워크가 요구하는 두가지 조건](#junit-프레임워크가-요구하는-두가지-조건)
      - [JUnit 테스트 실행](#junit-테스트-실행)
  - [개발자를 위한 테스팅 프레임워크 JUnit](#개발자를-위한-테스팅-프레임워크-junit)
    - [테스트 결과의 일관성](#테스트-결과의-일관성)
      - [동일한 결과를 보장하는 테스트](#동일한-결과를-보장하는-테스트)
    - [포괄적인 테스트](#포괄적인-테스트)
      - [getCount() 테스트](#getcount-테스트)
      - [addAndGet() 테스트 보완](#addandget-테스트-보완)
      - [get() 예외 조건에 대한 테스트](#get-예외-조건에-대한-테스트)
      - [테스트를 성공시키기 위한 코드의 수정](#테스트를-성공시키기-위한-코드의-수정)
      - [포괄적인 테스트](#포괄적인-테스트-1)
    - [테스트가 이끄는 개발](#테스트가-이끄는-개발)
      - [기능설계를 위한 테스트](#기능설계를-위한-테스트)
      - [테스트 주도 개발](#테스트-주도-개발)
    - [테스트 코드 개선](#테스트-코드-개선)
      - [@Before](#before)
      - [JUnit이 하나의 테스트 클래스를 가져와서 수행하는 방식](#junit이-하나의-테스트-클래스를-가져와서-수행하는-방식)
      - [픽스처](#픽스처)
  - [스프링 테스트 적용](#스프링-테스트-적용)
    - [테스트를 위한 애플리케이션 컨텍스트 관리](#테스트를-위한-애플리케이션-컨텍스트-관리)
      - [스프링 테스트 컨텍스트 프레임워크 적용](#스프링-테스트-컨텍스트-프레임워크-적용)
      - [테스트 클래스의 컨텍스트 공유](#테스트-클래스의-컨텍스트-공유)
      - [@Autowired](#autowired)
    - [DI와 테스트](#di와-테스트)
      - [테스트 코드에 의한 DI](#테스트-코드에-의한-di)
      - [테스트를 위한 별도의 DI 설정](#테스트를-위한-별도의-di-설정)
      - [컨테이너 없는 DI 테스트](#컨테이너-없는-di-테스트)
      - [DI를 이용한 테스트 방법 선택](#di를-이용한-테스트-방법-선택)
  - [학슽 테스트로 배우는 스프링](#학슽-테스트로-배우는-스프링)
    - [학습 테스트의 장점](#학습-테스트의-장점)
    - [학습 테스트 예제](#학습-테스트-예제)
      - [JUnit 테스트 오브젝트 테스트](#junit-테스트-오브젝트-테스트)
      - [스프링 테스트 컨텍스트 테스트](#스프링-테스트-컨텍스트-테스트)
    - [버그 테스트](#버그-테스트)
      - [필요성과 장점](#필요성과-장점)
## UserDaoTest 다시보기
### UserDaoTest
```java
public class UserDaoTest {
    public static void main(String[] args) throws SQLException {
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);

        User user = new User();
        user.setId("user");
        user.setName("jinho");
        user.setPassword("single");

        dao.add(user);

        System.out.println(user.getId() + " 등록 성공");

        User user2 = dao.get(user.getId());
        System.out.println(user2.getName());
        System.out.println(user2.getPassowrd);

        System.out.println(user2.getId() + " 조회 성공");
    }
}
```
#### 웹을 통한 DAO 테스트 방법의 문제점
웹을 통해서 테스트하게되면 DAO만 테스트하는 것이 아닌 요청을 보내고 응답하는 모든 계층과 컴포넌트가 해당 테스트에 영향을 줄 수 있기 때문에 비효율적이다. 

#### 작은 단위 테스트
작은 단위로 나누어(한 가지 관심사에 집중해서) 테스트를 하게 되면 영향을 미칠 수 있는 다른 요소들에 신경 쓰지 않아도 된다. 이를 `단위 테스트(Unit Test)`라고 한다. 유닛의 크기가 정해진 것은 아니지만 하나의 관심에 집중해서 효율적으로 테스트할만한 범위의 단위이다. 

통제할 수 없는 외부의 리소스에 의존하는 테스트는 단위 테스트가 아니다.

#### 자동 수행 테스트 코드
편리하기 때문에 자주 반복할 수 있다. 나중에 프로그램의 규모가 커지게 되면 이전에 개발한 단위에서도 테스트를 진행해서 버그를 찾아낼 수 있다. 

#### 지속적인 개선과 점진적인 개발을 위한 테스트
테스트 코드를 작성해놓으면 리팩토링해도 기존의 기능을 만족하는지 확인할 수 있다. 이렇게 확인할 수 있게 되면 리팩토링을 좀 더 자유롭게 할 수 있다. 

### UserDaoTest의 문제점
- 수동 확인 작업의 번거로움
  - 입력 데이터는 미리 작성되었지만 결과는 사람이 확인해야한다. 
- 실행 작업의 번거로움
  - main 메소드에 테스트가 작성되어있기 때문에 따로 테스트할 수도 없고 항상 테스트해야한다.

## UserDaoTest 개선
두가지 문제점을 개선해보자.

### 테스트 검증의 자동화
> "테스트란 개발자가 마음 편하게 잠자리에 들 수 있게 해주는 것"

### 테스트의 효율적인 수행과 결과 관리
#### JUnit 테스트로 전환
제어의 역전을 통해서 main함수와 오브젝트 생성 없이 테스트를 실행할 수 있다.

#### 테스트 메소드 전환
##### Junit 프레임워크가 요구하는 두가지 조건
1. 메소드가 public 으로 선언되어야한다.
2. 메소드에 @Test라는 어노테이션을 붙여야한다.

```java
import org.junit.Test;
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

public class UserDaoTest {
    @Test
    public vois addAndGet() throws SQLException {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);
        User user = new User();
        user.setId("hello JUnit");
        user.setName("jinho");
        user.setPassword("dragon");

        dao.add(user);

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(suer2.getPassword, is(user.getPassword()));
    }
}
```
#### JUnit 테스트 실행
```java
public static void main(String[] args){
    JUnitCore.main("springbook.user.dao.UserDaoTest"); // 실행할 테스트 클래스
}
```

테스트에서 assertThat으로 결과가 검증되니 못하면 java.lang.AssertionError가 발생한다.

## 개발자를 위한 테스팅 프레임워크 JUnit
> 최신 버전을 따로 공부하자

### 테스트 결과의 일관성
테스트를 실행하기 전에 DB를 초기화해야하는 경우가 있어 테스트를 반복해서 실행하면 에러가 발생할 수 있다. 테스트를 마치면 테스트를 수행하기 이전의 상태로 만들어주는 것이 좋다.

dao에 deleteAll()과 getCount() 메소드를 생성해서 테스트 결과에 일관성을 부여하자

```java
@Test
public void addAndGet() throws SQLException {
    // ...
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    User user = new User();
    user.setId("testId");
    user.setName("jinho");
    user.setPassword("testpw");

    dao.add(user);
    assertThat(dao.getCount(), is(1));

    User user2 = dao.get(User.getId());

    assertThat(user2.getName(), is(user.getName()));
    assertThat(user2.getPassword(), is(user.getPassword()));
}
```

#### 동일한 결과를 보장하는 테스트
단위 테스트는 항상 일관성있는 결과가 보장돼야 한다. 테스트를 실행하는 순서를 바꿔도 항상 동일한 결과가 보장되어야 한다.

### 포괄적인 테스트
> 테스트를 안 만드는 것도 위험한 일이지만, 성의 없이 테스트를 만드는 바람에 문제가 있는 코드인데도 테스트가 성공하게 만드는 건 더 위험하다.

#### getCount() 테스트
드디어 한번에 설정 가능한 생선자를 사용한다.

```java
@Test 
public void count() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext(
        "applicationContext.xml"
    );

    UserDao dao = context.getBean("userDao", UserDao.class);
    User user1 = new User("id1", "jinho", "testpw1");
    User user2 = new User("id2", "zzzinho", "testpw2");
    User user3 = new User("id3", "patrick", "testpw3");

    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.add(user1);
    assertThat(dao.getCount(), is(1));

    dao.add(user2);
    assertThat(dao.getCount(), is(2));

    dao.add(user3);
    assertThat(dao.getCount(), is(3));
}
```

#### addAndGet() 테스트 보완
get()이 파라미터로 주어진 id에 해당하는 데이터를 가져온 것인지 아무 데이터가 가져온 것인지 확인할 수 없었다.
```java
@Test
public void addAndGet() throws SQLException {
    // ...
    
    User user1 = new User("id1", "sponge bob", "square pants");
    User user2 = new User("id2", "patrick", "star");

    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.add(user1);
    dao.add(user2);
    assertThat(dao.getCount(), is(2));

    User userget1 = dao.get(user1.getId());
    assertThat(userget1.getName(), is(user1.getName()));
    assertThat(userget1.getPassword(), is(user1.getPassword()));

    User userget2 = dao.get(user2.getId());
    assertThat(userget2.getName(), is(user2.getName()));
    assertThat(userget2.getPassword(), is(user2.getPassword()));
}
```

#### get() 예외 조건에 대한 테스트
get() 메소드에 전달된 id 값에 해당하는 사용자 정보가 없는 경우도 생각해봐야한다. null을 반환하거나 예외를 던지는 방법이 있다. (Optional을 사용할 수도 있다.)

```java
@Test(expected=EmptyResultDataAccessException.class) // 테스트 중 발생할 예외 클래스 지정
public void getUserFailure() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.get("unknown_id"); // 여기서 예외 발생해야함
}
```

기존의 UserDao에서 쿼리 결과의 첫번째 로우를 가져오게하는 rs.next()를 실행할 때 가져올 로우가 없다면 SQLException이 발생할 것이다. 

#### 테스트를 성공시키기 위한 코드의 수정
```java
public User get(String id) throws SQLException {
    // ... 
    ResultSet rs = ps.executeQuery();

    User user = null;
    if(rs.next()){
        user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));
    }

    rs.close();
    ps.close();
    c.close();

    if(user == null) throw new EmptyResultDataAccessException(1);

    return user;
}
```

#### 포괄적인 테스트
> "항상 네거티브 테스트를 먼저 만들라" - 로드 존슨

### 테스트가 이끄는 개발
테스트를 먼저 만들어 테스트가 실패하는 것을 보고나서 코드를 작성했다. 테스트할 코드 없이 테스트 코드먼저 작성한 것이다. 이런 전략을 많이 사용되고 있다.

#### 기능설계를 위한 테스트
코드를 어떻게 테스트할까하면 테스트 코드를 작성한 것이 아닌 추가하고 싶은 기능을 코드로 표현하려했다.
getUserFailure() 테스트에는 만들고싶은 기능에 대한 `조건`과 `행위`,`결과`에 대한 내용이 잘 표현되어있다.


테스트 코드는 잘 작성된 기능정의서처럼 보인다. 그래서 보통 기능설계, 구현, 테스트라는 일반적인 개발 흐름의 기능설계에 해당하는 부분을 테스트 코드가 일부분 담당하고 있다고 볼 수 있다.

테스트에 실패하면 설계한 대로 코드가 만들어지지 않았음을 알 수 있다.

#### 테스트 주도 개발
기능을 명세하고 생성된 코드를 검증할 수 있도록 테스트 코드를 먼저 만들고 그 후에 코드를 작성하는 방식을 `테스트 주도 개발(TDD, Test Driven Development)`라고 한다. 

TDD는 개발자가 테스트를 먼저 만들며 개발하는 방법의 장점을 극대화한 방법이다. 

> "실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다" 

> 코드를 만들고 나서 시간이 많이 지나면 테스트를 만들기가 귀찮아 진다. 🥲

TDD에서는 테스트를 작성하고 이를 성공시키는 코드를 만드는 작업의 주기를 가능한 짧게 가져가도록 권장한다. 

> '눈물 젖은 커피와 함께 며칠간 밤샘을 하며 오류를 잡으려고 애쓰다가 전혀 생각지도 못했던 곳에서 간신히 찾아낸 작은 버그 하나의 추억'이라는 건, 사실 '진작에 충분한 테스트를 했었다면 쉽게 찾아냈을 것을 미루고 미루다 결국 커다랍 삽질로 만들어버린 어리석은 기억'일 뿐이다.

### 테스트 코드 개선
애플리케이션 코드만이 리팩토링의 대상은 아니다.

#### @Before
```java
import org.junit.Before;

public class UserDaoTest {
    private UserDao dao;

    @Before
    public void setUp(){
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        this.dao = context.getBean("userDao", UserDao.class);
    }

    // 테스트들
}
```

#### JUnit이 하나의 테스트 클래스를 가져와서 수행하는 방식
1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다.
2. 테스트 클래스의 오브젝트를 하나 만든다.
3. @Before가 붙은 메소드가 있으면 실행한다.
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
5. @After가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
7. 모든 테스트의 결과를 종합해서 돌려준다. 

기억해야할 점은 각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다는 점이다. 테스트 클래스가 @Test 테스트 메소드를 두개 가지고 있다면 테스트가 실행되는 중에 JUnit은 이 클래스의 오브젝트를 두번 만든다.

왜?

JUnit 개발자는 각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 확실히 보장해주기 위해 매번 새로운 오브젝트를 만들게 했다. 

#### 픽스처
테스트를 수행하는 데 필요한 정보나 오브젝트를 `픽스터(fixture)`라고 한다. 일반적으로 픽스처는 여러 테스트에서 반복적으로 사용되기 때문에 @Before 메소드를 이용해 생성해두면 편리하다.

```java
public class UserDaoTest {
    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        // userdao 생성 생략
        this.user1 = new User("1L", "jinho", "gogo");
        this.user2 = new User("2L", "spongebob", "square pants");
        this.user3 = new User("3L", "patrick", "star");
    }

    // 테스트들...
}
```

## 스프링 테스트 적용
지금까지 만든 테스트 코드를 사용하면 테스트 메소드가 실행될 때마다 애플리케이션 컨텍스트가 생성된다. 빈이 많아지고 복잡해지면 컨텍스트 생성에 많은 시간이 걸릴 수 있다. 애플리케이션 컨텍스트가 만들어질 때는 모든 싱글톤 빈 오브젝트를 초기화한다. 그렇기 때문에 가능한 독립적인 새로운 오브젝트를 만들어서 테스트하지만 애플리케이션 컨텍스트는 테스트 전체가 공유하는 오브젝트를 만들기도 한다. 

단, 테스트늘 일고나성있는 실행 결과를 보장해야하고 실행 순서가 결과에 영향을 미치지 않아야 한다. 

문제는 JUnit에서 제공하는 @BeforeClass를 사용하면 테스트 클래스 전체게 걸쳐서 한번만 실행되는 스태틱 메소드를 사용할 수 있다. 하지만 스프링이 직접 제공하는 애플리케이션 컨텍스트 테스트 지원 기능을 사용하는 것이 더 편리하다.

### 테스트를 위한 애플리케이션 컨텍스트 관리
#### 스프링 테스트 컨텍스트 프레임워크 적용
```java
@RunWtih(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
    @Autowired
    private ApplicationContext context;

    @Before 
    public void setUp() {
        this.dao = this.context.getBean("userDao", UserDao.class);
        // ...
    }
}
```

`@RunWith`는 JUnit 프레임워크의 테스트 실행 방법을 확장할 때 사용 한다.

`SpringJUnit4ClassRunner`라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정해주면 테스트를 진행하는 중 테스트가 사용할 컨텍스트를 만들고 관리하는 작업을 진행해준다. 

#### 테스트 클래스의 컨텍스트 공유
여러 개의 테스트 클래스에서도 컨텍스트 공유가 가능하다. 
```java
@RunWtih(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
    // ...
}
```
```java
@RunWtih(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class GroupDaoTest {
    // ...
}
```
같은 컨텍스트 파일을 사용하여 공유 가능

#### @Autowired
`@Autowired`가 붙은 인스턴스 변수가 있으면 테스트 컨텍스트 프레임워크는 변수타입과 일치하는 컨텍스트 내의 빈을 찾는다. 별도의 DI 설정 없이 필요한 타입정보를 이용해 빈을 자동으로 가져올 수 있다. 

앞에서 만든 테스트 코드에서 `ApplicationContext` 타입 변수에 `@Autowired`를 붙였는데 어떻게 DI 됐을까? 

스프링 애플리케이션 컨텍스트는 초기화할 때 자기 자신도 빈으로 등록한다. 따라서 DI 가능.

```java
public class UserDaoTest {
    @Autowired
    UserDao dao;
    // ...
}
```

만약 같은 타입의 빈이 두개 이상 설정되어있다면 가져올 수 없다. 

### DI와 테스트
만약 절대로 구현체를 변경하지 않는 경우에도 인터페이스를 사용해야할까? 저자는 인터페이스를 두고 DI를 적용해야한다고 한다.

1. 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없다.
2. 클래스의 구현 방식이 빠귀지 않아도 인터페이스를 두고 DI를 적용하면 다른 차원의 서비스 기능을 도입할 수 있다.
3. 테스트를 하기 쉽다.

#### 테스트 코드에 의한 DI
운영도 데이터 베이스로 테스트할 수는 없으니까 DI를 사용해서 테스트용 데이터베이스로 변경해주자. DAO가 사용할 DataSource를 변경해주는 방법으로 변경이 가능하다. 

```java
@DirtiesContext // 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에 알려준다.
public class UserDaoTest {
    @Autowired
    UserDao dao;
    
    @Before
    public void setUpt() {
        // ...
        DataSource dataSource = new SingleConnectionDataSource(
            "jdbc:mysql://localhost/testdb", "spring", "book", true
        );
        dao.setDataSource(dataSource);
    }
    // ...
}
```

해당 방식은 이미 애플리케이션 컨텍스트에서 applicationContext.xml 파일의 설정정보를 따라 구성한 오브젝트를 강제로 변경했기 때문에 주의해서 사용해야한다. `@DirtiesContext`를 사용해서 스프링의 테스트 컨텍스트 프레임워크에게 해당 클래스의 테스트에서 애플리케이션 컨텍스트의 상태를 변경한다는 것을 알려준다. 테스트 컨텍스트는 이 어노테이션이 붙은 테스트 클레스에는 애플리케이션 컨텍스트를 공유하지 않는다. 테스트 중에 변경한 컨텍스트가 뒤의 테스트에 영향을 주지않게 하기 위함이다. 

#### 테스트를 위한 별도의 DI 설정
테스트 코드에서 직접 DI를 하는 방법은 단점이 많다. 코드가 많아저 번거롭고 애플리케이션 컨텍스트도 매번 새로 만들어야한다. 테스트에서 사용할 전용 설정파일을 따로 만들어두면 이전 방식에서 장점을 사용할 수 있다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
    // ...
}
```

#### 컨테이너 없는 DI 테스트
DI를 테스트에 이용하는 다른 방법으로 스프링 컨테이너를 사용하지 않고 테스트하는 방법이 있다. 

```java
public class UserDaoTest {
    UserDao dao; // @Auwotired가 없다.

    @Before
    public void setUp() {
        // ...
        dao = new UserDao();
        DataSource dataSource = new SingleConnectionDataSource(
            "jdbc:mysql://localhost/testdb", "spring", "book", true
        );
        dao.setDataSource(dataSource);
    }
    //...
}
```

테스트를 위한 DataSource를 직접만드는 번거로움은 있지만 애플리케이션 컨텍스트를 사용하지 않기 때문에 코드는 더 단순해지고 이해하기 편해졌다. 

> 어디에 DI를 적용할지 고민된느 경우 효과적인 테스트를 만들기 위해서는 어떤 필요가 있을지를 생각해보면 도움이 된다.

#### DI를 이용한 테스트 방법 선택
3가지 방법 중에 어떤 방법이 좋을까? 일단 스프링 컨테이너 없이 테스트할 수 있는 방법을 가장 우선적으로 고려하자. 해당 방법이 테스트 수행 속도가 가장 빠르고 테스트가 간결하다. 

여러 오브젝트와 복잡한 의존관계를 갖고 있는 오브젝트를 테스트해야할 경우네는 스프링의 설정을 이용한 DI 방식의 테스트가 편리하다. 테스트 전용 설정 파일을 따로 만들어 사용하는 것이 좋다. 

예외적으로 강제로 의존관계를 구성해서 테스트하는 경우에는 수동으로 DI 해서 테스트 하자.

## 학슽 테스트로 배우는 스프링
학습 테스트를 통해 자신이 사용할 API나 프레임워크의 기능을 테스트로 보면서 사용방법을 익힐 수 있다.

### 학습 테스트의 장점
- 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.
- 학습 테스트 코드를 개발 중에 참고할 수 있다.
- 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.
- 테스트 작성에 대한 좋은 훈련이 된다.
- 새로운 기술을 공부하는 과정이 즐거워진다!

### 학습 테스트 예제
#### JUnit 테스트 오브젝트 테스트
JUnit은 테스트 메소드를 수행할 때마다 새로운 오브젝트를 만든다. 확인해봤냐? 확인해보자.

새로운 테스트 클래스를 만들고 적당한 이름으로 세 개의 테스트 메소드를 추가한다. 테스트 클래스를 자신의 타입으로 스태틱 변수를 하나 선언한다. 테스트 메소드마다 현재 스태틱 변수에 담긴 오브젝트와 자신을 비교해서 같지 않다는 것을 확인한다. 그리고 현재 오브젝트를 그 스태틱 변수에 저장한다.

```java
public class JUnitTest {
    static Set<JUnitTest> testObjects = new HashSet<>();

    @Test public void test1() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);
    }

    @Test public void test2() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);
    }

    @Test public void test3() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);
    }
}
```

해당 테스트 코드로 매번 새로운 테스트 오브젝트가 생성된다는 것을 확인할 수 있다.

#### 스프링 테스트 컨텍스트 테스트
스프링의 테스트용 애플리케이션 컨텍스트는 테스트 개수에 상관없이 한개만 만들어진다. 확인해보자.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class JUnitTest {
    @Autowired ApplicationContext context;

    static ApplicationContext contextObject = null;

    @Test public void test1() {
        assetThat(contextObject == null || contextObject == this.context, is(true));
        contextObject = this.context;
    }

    @Test public void test2() {
        assetTrue(contextObject == null || contextObject == this.context);
        contextObject = this.context;
    }

    @Test public void test3() {
        assertThat(contextObject, either(is(nullValue())).or(is(this.context)));
        contextObject = this.context;
    }
}
```

### 버그 테스트
디버깅을 무턱대고 코드를 뒤지는 것보다 버그 테스트를 만들어보는 편이 유용하다.

버그 테스트는 일단 실패하도록 만들어야한다. 버그가 원인이 돼서 테스트가 실패하는 코드를 만들고 버그 테스트가 성공하도록 애플리케이션 코드를 수정한다. 테스트가 성공하면 버그는 해결된 것이다.

#### 필요성과 장점
- 테스트의 완성도를 높여준다.
- 버그의 내용을 명확하게 분석하게 해준다.
- 기술적인 문제를 해결하는 데 도움이 된다.