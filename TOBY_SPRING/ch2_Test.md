# Test
저자는 스링프링이 제공하는 가장 큰 가치는 객체지향과 테스트라고 했다. 
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

