# XML을 이용한 설정
- [XML을 이용한 설정](#xml을-이용한-설정)
  - [XML 설정](#xml-설정)
    - [conectionMaker() 전환](#conectionmaker-전환)
    - [userDao() 전환](#userdao-전환)
    - [XML 의존관계 주입 정보](#xml-의존관계-주입-정보)
  - [XML을 이용하는 애플리케이션 컨텍스트](#xml을-이용하는-애플리케이션-컨텍스트)
  - [DataSource 인터페이스로 변환](#datasource-인터페이스로-변환)
    - [DataSource 인터페이스 적용](#datasource-인터페이스-적용)
  - [프로퍼티 값의 주입](#프로퍼티-값의-주입)
    - [값 주입](#값-주입)
    - [value 값의 자동 변환](#value-값의-자동-변환)

> 스프링은 자바 클래스를 이용하는 방범 외에 XML과 같은 다양한 방법으로 DI를 사용할 수 있다.

## XML 설정
- 빈의 이름: @Bean 메소드 이름, getBean()에서 사용됨
- 빈의 클래스: 빈 오브젝트를 어떤 클래스를 이용해서 만들지 정의
- 빈의 의존 오브젝트: 빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣어준다. 

### conectionMaker() 전환

|               | 자바 코드 설정정보        | XML 설정정보                  |
|---------------|---------------------------|-------------------------------|
| 빈 설정 파일  | @Configuration            | \<bean\>                      |
| 빈의 이름     | @Bean methodName()        | \<bean id="methodName"\>      | 
| 빈의 클래스   | return new BeanClass();   | class="a.b.c...BeanClass\>    |

 ```xml
 <bean id="connectionMaker" class="springbook...NConnectionMaker" />
 ```

 ### userDao() 전환
 - ref는 수정자 메소드를 통해 주입해줄 오브젝트의 빈 이름
```xml
<bean id="userDao" class="springbook.dao.UserDao">
    <property name="connectionMaker" ref="connectinoMaker" />
</bean>
```
### XML 의존관계 주입 정보
```xml
<beans>
    <bean id="connectionMaker" class="springBook.user.dao.NConnectionMaker"/>
    <bean id="userDao" class="springbook.user.dao.UserDao"> 
        <property name="connectionMaker" ref="connectionMaker" />
    </bean>
</beans>
```
- property의 name은 메소드의 이름이고 ref는 빈의 이름이다.

- 인터페이스를 구현한 여러개의 의존 오브젝트를 정의해두고 원하는 것을 가져와 DI 할 수 있다.
```xml
<beans>
    <bean id="localDBConnectionMaker" class="LocalDBConnectionMaker" />
    <bean id="testDBConnectionMaker" class="TestDBConnectionMaker" />
    <bean id="productDBConnectionMaker" class="ProductDBConnectionMaker" />

    <bean id="userDao" class="springbook.user.dao.UserDao">
        <property name="connectionMaker" ref="localDBConnectionMaker" />
    </bean>
</beans>
```

## XML을 이용하는 애플리케이션 컨텍스트
- XML에서 빈의 의존관계 정보를 이용하는 IoC/DI 작업에는 `GenericXmlApplicationContext`를 사용
- GenerticXmlApplicationContext의 생성자에 xml 파일을 클래스 패스를 지정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org./schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    
    <bean id="connectionMaker"  class="springbook.user.dao.DConnectionMaker" />

    <bean id="userDao" class="springbook.user.dao.UserDao">
        <property name="connectionMaker" ref="connectionMaker" />
    </bean>
</beans>
```
- UserDaoTest의 AnnotaionConfigApplicationConext 수정
```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
```

## DataSource 인터페이스로 변환
### DataSource 인터페이스 적용
대부분의 실사용환경에서는 DataSource를 사용해서 DB 커넥션을 생성해준다. 예제를 DataSource로 리팩토링해보자

- DataSource 인터페이스
```java
public interface DataSource extends CommonDataSource, Wrappre {
    Connection getConnection() throws SQLException;
    // ...
}
```
- DataSource를 적용한 UserDao
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource){
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        Connection c = dataSource.getConnection();
        // ... 
    }
    // ...
}
```
- Java 코드 설정 방식
```java
@Bean
public DataSource() {
    SimpleDriverDateSource dataSource = new SimpleDriverDataSource();
    
    dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
    dataSource.setUrl("jdbc:mysql://localhost/springbook");
    dataSource.setUsername("spring");
    dataSource.setPassword("book");

    return dataSource;
}
```
- DataSource 타입의 빈을 DI 받는 useDao() 빈 정의 메소드
```java
@Bean
public UserDao userDao() {
    UserDao userDao = new UserDao();
    userDao.setDataSource(dataSource());
    return userDao();
}
```
- XML 설정 방식
```xml
<bean id="dataSrouce" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" />
```
-> SimpleDriverDataSource의 오브젝트는 만들었지만 DB 접속정보는 나타나 있지 않다.
-> 값을 주입해줘야한다.

## 프로퍼티 값의 주입
### 값 주입
```xml
<property name="driverClass" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost/springbook" />
<property name="username" value="spring"/>
<property name="password" value="book" />
``` 

### value 값의 자동 변환
driveClass를 보면 오브젝트가 아닌 스트링 값이 입력된 것을 알 수 있다. 스프링에서는 해당 스트링 값을 적절한 형태로 변환해준다. 