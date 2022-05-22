# 오브젝트와 의존관계
- [오브젝트와 의존관계](#오브젝트와-의존관계)
  - [DAO(Data Access Object)](#daodata-access-object)
    - [Entity](#entity)
    - [DAO](#dao)
    - [Test](#test)
  - [DAO의 분리](#dao의-분리)
    - [관심사의 분리](#관심사의-분리)
    - [커넥션 만들기의 추출](#커넥션-만들기의-추출)
      - [UserDao의 관심사항](#userdao의-관심사항)
      - [중복 코드의 메소드 추출](#중복-코드의-메소드-추출)
    - [DB 커넥션 만들기의 독립](#db-커넥션-만들기의-독립)
      - [상속을 통한 확장](#상속을-통한-확장)
        - [디자인 패턴](#디자인-패턴)
        - [부족한 점](#부족한-점)
  - [DAO의 확장](#dao의-확장)
    - [클래스의 분리](#클래스의-분리)
    - [인터페이스의 도입](#인터페이스의-도입)
      - [인터페이스를 통한 관심사 분리된 UserDao](#인터페이스를-통한-관심사-분리된-userdao)
    - [관계 설정 책임의 분리](#관계-설정-책임의-분리)
    - [원칙과 패턴](#원칙과-패턴)
      - [개방 폐쇄 원칙(OCP, Open-Close Principle)](#개방-폐쇄-원칙ocp-open-close-principle)
      - [높은 응집도와 낮은 결합도(High Coherence and Low Coupling)](#높은-응집도와-낮은-결합도high-coherence-and-low-coupling)
      - [전략 패턴(Strategy Pattern)](#전략-패턴strategy-pattern)
## DAO(Data Access Object)
> DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하는 오브젝트
- 자바빈(Java Bean)
  - 자바빈은 다음 두가지 관례를 따라 만들어진 오브젝트이다.
  - 디폴트 생성자
    - 자바빈은 파라미터가 없는 디폴트 생성자를 갖고 있어야한다. 
    - 리플렉션을 이용해 오브젝트를 생성하기 때문
  - 프로퍼티
    - 자바빈이 노출하는 이름을 가진 속성
    - setter와 getter를 이용해 수정 또는 조회할 수 있다.
  
### Entity
```java
@Getter
@Setter
class User {
    String id;
    String name;
    String password;
}
```
```sql
create table users (
    id varchar(10) primary key,
    name varchar(20) not null,
    password varchar(10) not null
)
```
### DAO
- JDBC를 이용하는 작업의 일반적인 순서
  - DB 연결을 위한 Connection을 가져온다.
  - SQL을 담은 Statement를 만든다.
  - 만들어진 Statement를 실행한다.
  - 조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트에 옮겨준다.
  - 작업 중에 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫아준다.
  - JDBC API가 만들어내는 예외를 직접 처리하거나, thorws를 통해 메소드 밖으로 던진다.
    - 예외는 모두 메소드 밖으로 던지는 편이 간단하다.

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
            "jdbc:mysql://localhost/springbook", "spring", "book"
        );
        
        PreparedStatement ps = c.prepareStatement(
            "insert into users(id, name, password) values(?,?,?)"
        );

        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
            "jdbc:msql://localhost/springbook", "spring", "book"
        );

        PreparedStatement ps = c.prepareStatement(
            "select * from users where id = ?"
        );
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getName("name"));
        user.setPassword(rs.getPassword("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```
- DAO는 잘 작동하겠지만 잘 작성된 코드는 아니다.

### Test
```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao dao = new UserDao();

    User user = new User();
    user.setId("zzzinho");
    user.setName("진호");
    user.setPassowrd("not_married");

    dao.add(user);

    System.out.println(user.getId() + " 등록 성공");

    User user2 = dao.get(user.getId());
    System.out.println(user2.getName());
    System.out.println(user2.getPassword());

    System.out.println(user2.getId() + " 조회 성공");
}
```

## DAO의 분리
### 관심사의 분리
> 관심이 같은 것끼리  하나의 객체 안으로 또는 친한 객체로 모이게하고 관심이 다른 것은 가능한 따로 떨어져서 서로 영향을 주지 않도록 분리하는것

### 커넥션 만들기의 추출
#### UserDao의 관심사항
- add() 메소드에서 3가지 관심사항을 추출할 수 있다.
  - DB와 연결을 위한 커넥션을 어떻게 가져올까 
  - 사용자 등록을 위해 DB에 보낼 SQL 문장을 담을 Statement 관리
  - 작업이 끝나면 사용한 리소스를 닫아줘서 공유 리소스를 시스템에 돌려준다.
#### 중복 코드의 메소드 추출
```java
private Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection(
        "jdbc:mysql://localhost/springbook", "spring", "book");
    );
    return c;
}
```
- Connection을 형성하는 부분을 추출해서 중복을 줄인다.
  -  `메소드 추출(extract method)`하여 리팩토링

### DB 커넥션 만들기의 독립
N사와 K사에 UserDao를 판매한다고 하자. 지금은 데이터베이스가 고정적으로 작성되어있기 때문에 다양한 데이터 베이스에 연결 할 수 있어야한다. 소스코드를 직접 제공할 수 도 있지만 코드가 아닌 컴파일된 파일을 통해서도 연결할 수 있어야한다.

#### 상속을 통한 확장
```java
public abstract class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
        ...
    }
    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
        ...
    }
    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
```
```java
public class NUserDao extends UserDao {
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        // N 사 DB connection 생성 코드
    }
}
```
```java
public class KUserDao extends UserDao {
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        // K 사 DB Connection 생성 코드
    }
}
```
- DAO의 핵심 기능인 데이터 처리 방식은 유지하면서 데이터베이스 연결만 분리하였다.
- `템플릿 메소드 패턴(template method pettern)`: 슈퍼 클래스에 기본적인 로직의 흐림을 만들고, 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브 클래스에서 필요에 맞게 구현해 사용하도록 하는 패턴

##### 디자인 패턴
- 특정 상황에서 자주 만나는 문제를 해결하기 위한 재사용 가능한 솔류션
- 어떤 패턴을 적용하냐에따라 설계 의도와 해결책을 함께 설명할 수 있다. 
- 클래스 상속 또는 오브잭트 합성 두가지가 있다.
- GoF의 디자인 패턴을 추천한다고 한다. 나중에 읽어봐야겠다.

> ##### 템플릿 메소드 패턴
> - 상속을 통해 슈퍼 클래스의 기능을 확장할 때 사용
> - 변하지 않는 기능을 슈퍼 클래스에서 정의
> - 확장하는 기능은 서브 클래스에서 작성
> ```java
> public abstract class Super {
>   public void templateMethod() {
>        // 기본 알고리즘
>        hookMethod();
>        abstractMethod();
>        ...
>    }
>    protected void hookMethod() {} -> 선택적으로 오버라이드 가능한 훅 메소드
>    public abstract void abstractMethod(); -> 서브 클래스에서 반드시 구현해야하는 추상 메소드
>}
>```
>```java
>public class Sub1 extends Super {
>    protected void hookMethod () {
>        ...
>    }
>    public void abstractMethod() {
>        ...
>    }
>}
>```

> ##### 팩토리 메소드 패턴
> - 상속을 통해 기능을 확장하는 패턴
> - 슈퍼클래스 코드에서는 서브 클래스에서 구현할 메소드를 호출해서 필요한 타입의 오브젝트를 가져와서 사용
> - 주로 인터페이스 타입으로 오브젝트를 리턴 -> 서브 클래스에서 정확히 어떤 클래스의 오브젝트를 > 만들어리턴할지는 슈퍼클래스에서는 알지 못한다.

템플릿 메소드 패턴 또는 팩토리 메소드 패턴으로 관심사항이 다른 코드를 분리하고 독립적으로 변경 또는 확장 할 수 있도록 만들 수 있다.

##### 부족한 점
- 이미 UserDao가 다른 목적을 위해 상속을 사용하고 있다면 다중 상속이 안되는 자바의 특성상 어렵다.
- 상소을 통상 상하위 클래스의 관계는 생각보다 밀접하다.
- 슈퍼 클래스 내부의 변경이 있을 때 모든 서브 클래스를 수정하거나 다시 개발해야할 수도 있다.
- 확장 기능은 DB 커넥션 생성 기능을 다른 DAO에는 적용할 수 없다.

## DAO의 확장
> 모든 오브젝트는 변한다.

- 서로 영향을 주지 않고 필요한 시점에 독립적으로 변경할 수 있어야한다.
- 상속은 여러가지 단점이 많다.

### 클래스의 분리
- `관심사`가 다르고 `변화의 성격`이 다른 코드를 분리하자.
- DB 커넥션과 관련된 부분을 서브 클래스가 이날 별도의 클래스에 담는다.

```java
public class UserDao {
    private SimpleConnecitonMaker simipleConnectionMaker;

    public UserDa(){
        simpleConnecitonMaker = new SimpleConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.makeNewConnection();
        ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Conneciton c = simpleConnectionMaker.makeNewConnection();
        ...
    }
}
```
- SimpleConnectionMaker
```java
public class SimpleConnectionMaker {
    public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
            "jdbc:myqsl://loaclhost/springbook", "spring", "book");
        
        return c ;
    }
}
```
- 성격에 따라 분리하였지만 N사와 K사로 확장이 불가능해졌다.
- `SipleConnectionMaker`가 너무 많이 알고있어야한다.

### 인터페이스의 도입
- 두개의 클래스가 강하게 연결되어 있지 않도록 중간에 추상적인 느슨한 연결 고리를 만들어 주면 해결할 수 있다.
- 인터페이스는 자신을 구현한 클래스에 대한 구체적인 정보는 모두 감춘다.
- 인터페이스는 어떤 일을 하겠다는 기능만 정의해놓은 것
  
```java
public interface ConnectionMaker {
    public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

```java
public class KConnectionMaker implements ConnectionMaker {
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        ...
    }
}
```
#### 인터페이스를 통한 관심사 분리된 UserDao
```java
public class UserDao {
    private ConnectionMaker connectionMaker;

    public UserDao() {
        connecitonMaker = new KConnectionMaker();
    }
    public void addUser(User user) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
        ...
    }
    public void getUer(String id) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
        ...
    }
}
```
- 인터페이스를 사용해서 DB 커넥션을 제공하는 구체적인 정보는 제거했다.
- 하지만 어떤 클래스의 오브젝트를 사용할지 결정하는 생성자 코드는 그대로다.

### 관계 설정 책임의 분리
- UserDao와 ConnectionMaker라는 두개의 관심을 인터페이스를 사용하여 분리했지만 UserDao는 인터페이스와 구체적인 클래스까지 알아야한다.
- UserDao에는 어떤 클래스를 사용해서 커넥션을 제공받는지의 관심사항이 있다. -> 분리해야함
- 오브젝트 사이의 관계를 만들기 위해서 직접 생성할 수도 있지만 외부에서 만들어진 것을 가져오는 방법도 있다.
- 최종 목표는 UserDao의 수정 없이 DB 커넥션을 바꿀 수 있어야한다.
- 객체지향의 다형성을 통해서 런타입 중에 관계를 생성해줄 수 잇다.

```java
public UserDao(ConnectionMaker connectionMaker){
    this.connectionMaker = connectionMaker;
}
```
- UserDao의 클라이언트에 책임을 넘겨서 클라이언트가 사용할 ConnectionMaker를 넣어줄 수 있다.

```java
public class UserDaoTest {
    public static void main(String[] args) throws ClassNotFoundException {
        ConnectionMaker connectionMaker = new KConnectionMaker();

        UserDao dao = new UserDao(connectionMaker);
        ...
    }
}
```

### 원칙과 패턴
#### 개방 폐쇄 원칙(OCP, Open-Close Principle)
- 클래스나 모듈은 확장에는 열려 있어야하고 변경에는 닫혀있어야한다.
- UserDao는 DB 연결 방법 확장에는 열려 있고 UserDao에는 영향이 가지 않는다.

#### 높은 응집도와 낮은 결합도(High Coherence and Low Coupling)
- 높은 응집도
  - 하나의 모듈 클래스가 하나의 책임 또는 관심사에만 집중
  - 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다.
- 낮은 결합도
  - 하나의 오브젝트가 변경이 일어날 때 관계를 맺고 있는 다른 옵젝트에게 변화를 요구하는 정도가 낮다.
  - 책인과 관심사가 다른 오브젝트 또는 모듈과는 느슨한 연결 형태를 유지해야한다.

#### 전략 패턴(Strategy Pattern)
- 자신의 기능 맥락(context)에서 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 외부로 분리시키고 구현한 알고리즘을 필요에 따라 바꿔 쓸 수 있게하는 디자인 패턴
