# IoC & DI
## IoC의 종류
1. 의존성 lookup
   1. 의존성 풀(dependency pull)
   2. 문맥에 따른 의존성 룩업(contextualized Denpendency Lookup)
2. 의존성 주입(Dependency Injection)
   1. 생성자 의존성 주입
   2. 수정자 의존성 주입

### Dependency Pull
의존성 풀에서는 필요에 따라 레지스트리에서 의존성을 가져온다. 
```java
public class DependencyPull{
    public static void main(String... args){
        ApplicationContext ctx = new ClassPathXmlApplicationContext("spring/app-context.xml");
        MessageRenderer mr = ctx.getBean("renderer", MessageRenderer.class);
        mr.render();
    }
}
```
위 코드에서는 xml파일에서 의존성을 읽어와서 MessageRenderer 인스턴스를 생성해준다.
### Contextualized Dependency Lookup(CDL)
CDL은 의존성 풀처럼 특정 중앙 레지스트리에서 의존성을 가져오는 것이 아닌 자원을 관리하는 컨테이너에서 의존성을 가져온다. CDL은 항상 수행되는 것이 나닌 정해진 시점에 수행하게 된다.
```java
public interface ManageedComponent {
    void performLookup(Container container);
}
```
컴포넌트는 해당 인터페이스를 구현해서 의존 관계를 얻으려는 컨테이너에 신호를 보낸다. 일반적으로는 톰캣(Tomcat)이나 제이보스(JBoss)와 같은 기반 애플리케이션 서버나 기발 프레임워크, 스프링과 같은 애플리케이션 프레임워크에서 제공한다. 
```java
public interface Container {
    Object getDependency(String key);
}
```
컨테이너가 컴포넌트에 의존성을 전달할 준비가 되면 컨테이너는 차례대로 perfromLookup() 메서드를 호출한다.
```java
public class ContextualizedDependencyLookup implements MnagedComponent {
    private Dependency dependency;

    @Override void performLookup(Container container){
        this.dependency = (Dependency) container.getDependency("myDependency");
    }
}
```
### 생성자 의존성 주입
Constructor를 사용해서 컴포넌트가 필요로하는 의존성을 제공해 줄 수 있다. IoC 컨테이너는 해당 컴포넌트를 초기화할 때 컴포넌트에 필요한 의존성을 전달한다.
```java
public class ConstructorInjection{
    private Dependency dependency;

    public ConstructorInjection(Dependency dependency){
        this.dependency = dependency;
    }
}
```
> 추천되는 IoC 방법으로 서로 참조하는 것을 방지해주는 장점도 가지고 있다.

### 수정자 의존성 주입
Setter를 사용해서 의존성을 주입해줄 수 있다. 수정자 주입을 사용하면 의존성 없이도 객체를 생성하여 나중에 의존성을 주입해줄 수 있다. 
```java
public class SetterInjection {
    private Dependency dependency;

    public void setDependency(Dependency dependency){
        this.dependency = dependency;
    }
}
```
### @Autowired(Field Injection)
@Autowired 어노테이션을 사용하여 의존성 주입을 해주는 것도 가능하다.
```java
public class FieldInjection {
    @Autowired
    private Dependency dependency;
}
```
간편하다는 장점이 있지만 안정적이지 않다.

> 의존성 주입에서 어떤 방법이 좋은지는 아래의 포스팅을 확인하면 유용하다.
> 
> [[생성자 주입을 사용해야하는 이유 - YABOONG]](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)

## 의존성 주입 vs 의존성 룩업
의존성 주입을 사용하는 것이 좋은 선택이다. 의존성 룩업은 레지스트리를 참조하여 상호작용이 필요하지만 의존성 주입은 클래스는 생성자나 수정자를 통해서 의존성이 주입되기만 하면된다.

## 생성자 주입 vs 수정자 주입
### 생성자 주입
- 컴포넌트에 필요한 모든 의존성 제공을 보장
- 불변 객체를 설계 가능
### 수정자 주입
- 인터페이스에 모든 의존성을 선언 가능
- 구성정보를 주입하여 유동적인 사용 가능

## Bean & Bean Factory
- Bean: 컨테이너가 관리하는 모든 컴포넌트를 의미
- BeanFactory: 컴포넌트의 라이프사이클과 의존성을 관리

스프링의 ApplicationContext는 웹 컨테이너가 웹 애플리케이션을 시작하는 중에 web.xml 디스크립터 파일에서 선언된 ContextLoaderListener 클래스를 이용해서 부트스트랩한다.

BeanFactory를 구성할 때 대부분 XML 또는 Properites를 사용하여 외부에서 생성한다.
### BeanFactory
```java
public interface Oracle{
    String defineMeaningOfLife();
}
```
```java
@Setter
public class BookwormOracle implements Oracle {
    private Encyclopedia encyclopedia;

    @Override
    public String defineMeaningOfLife(){
        return "Encyclopedias are a waste of money - go see the world instead";
    }
}
```
Lombok의 Setter를 사용해서 수정자 의존성 주입을 해주었다.
#### BeanFactory 초기화 후 oracle bean 가져오기
```java
public class XmlConfigWithBeanFactory{
    public static void main(String... args){
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader rdr = new XmlBeanDefinitionReader(factory);
        rdr.loadBeanDefinitions(new ClassPathResource("spring/sml-bean-factory-config.xml"));
        Oracle oracle = (Oracle)facotry.getBean("oracle");
        System.out.println(oracle.defineMeaningOfLife());
    }
}
```
XmlBeanDefinitionReader를 사용해서 XML 파일의 BeanDefinition 정보를 읽어왔다. 
```xml
<bean id="oracle" name="wiseworm" class="com.example.di.BookwormOracle"/>
```
## ApplicationContext
스프링의 ApplicationContext 인터페이스는 BeanFactory를 상속한 인터페이스이다. ApplicationContext를 DI 서비스 외에도 드랜잭션 서비스, AOP 서비스, 국제화를 위한 메시지 소스, 애플리케이션 이벤트 처리와 같은 여러 서비스를 제공한다. 

### 스프링 컴포넌트 선언
```java
public interface MessageRenderer {
    void render();
    void setMessageProvider(MessageProvider provider);
    MessageProvider getMessageProvider();
}
```
```java
public class StandardOutMessagRenderer implements MessageRenderer{
   private MessagePorvider messageProvider;

   @Override
   public void renderer() {
       if(messageProvicer == null) throw new RuntimeException(
           "messagePoriver is in need: "
           + StandardOutputMessageRenderer.class.getName();
       )
       System.out.println(messageProvider.getMessage());
   } 

    @Override
    public void setMessageProvider(MessageProvider provider){
        this.messageProvider = provider;
    }

    @Override
    public MessageProvider getMessageProvider(){
        return this.messageProvider;
    }
}
```
```java
public interface MessageProvider{
    String getMessage();
}
```
```java
public class HelloWorldMessageProvider implements MessageProvider {
    @Override
    public String getMessage(){
        return "hell world";
    }
}
```
#### bean 정의 선언
```xml
<bean id="renderer"
    class "com.example.di.decoupled.StandardMessageRenderer"
    p:messageProvider-ref="provider"/>

<bean id="provider"
    class="com.exampel.di.decoupled.HelloWorldMessageProvider"/>
```
#### 어노테이션을 사용한 빈 정의 생성(간단)
```java
@Component
public class HellowordMessageProvider implements MessageProvider {
    @Overrid
    public String getMessage(){
        return "hello world";
    }
}
```
#### 어노테이션을 사용한 빈 정의 생성(복잡)
```java
@Service("redenrer")
public class StandardOutMessageRenderer implements MessaegRenderer {
    private MessageProvider messageProvider;

    @Override
    public void render(){
        if(messageProvider == null) throw new RuntimeError(
            "messageProvider is in need: "
            + StandardOutMessageRenderer.class.getName());
        )
    }

    @Override
    @Autowired
    public void setMessageProvider(MessageProvider messageProvider){
        this.messageProvider = messageProvider;
    }

    @Override
    public MessageProvider getMessageProvider(){
        return this.messageProvider;
    }
}
```
#### 컴포넌트 스캐닝 설정을한 구성 파일
```xml
<context:component-scan 
    base-package="com.example.di.annotated"/>
```
#### ApplicationContext로부터 빈 획득하기
```java
public class DeclareSpringCompoents {
    public static void main(String... args){
        GenericXmlApplicationContext ctx new GenerricXmlApplicationContext();
        ctx.load("classpath:spring/app-context-xml.xml");
        ctx.refresh();
        MessageRenderer messagRenderer ctx.getBean("renderer", MessageRenderer.class);
        messageRenderer.render();
        ctx.close();
    }
}
```
#### java configuration 사용
```java
@Configuration
public class HelloWrorldConfiguration {
    @Bean
    public MessageProvider provider(){
        return new HelloWorldMessaegProvider();
    }

    @Bean
    public MessageRenderer renderer(){
        MessageRenderer renderer = new StandardOutMessageRenderer();
        renderer.setMessageProvider(provider());
        return renderer;
    }
}
```
- AnnotationConfigApplicationContext를 사용해 빈 가져오기
```java
public class HelloWroldSpringAnnotated {
    public static void main(String... args){
        ApplicationContxt ctx = new AnnotaionConfigApplicationContext(HelloWorldConfiguration.class);
        MessageRenderer mr = ctx.getBean("redenderer", MessageRenderer.class);
        mr.render();
    }
}
```
- @ComponentScan이 적용된 구성 클래스
```java
@ComponentScan(basePackage={"com.example.di.annotated"})
@Configuration
public class HelloWroldConfiguration {

}
```
AnnotaionConfigApplicationContext를 사용해서 스프링 환경을 부트스트랩하는 기존 코드를 변경하지 않고 해당 클래스를 사용할 수 있다. 
- @ImportResource가 적용된 구성 클래스
```java
@ImportResouce(locations = {"classpath:spring/app-context-xml.xml"})
@Configuration
public class HelloWorldConfiguration{

}
```
#### 수정자 주입
- property 태그를 사용한 수정자 의존성 주입 
xml을 이용한 수정자 주입을 구성하기 위해서 의존성을 조입할 \<property\> 태그를 지정한다.
```xml
<bean id="renderer"
    class="com.example.di.decoupled.StandardOutMessageRenderer">
    <property name="messageProvider" ref="provider"/>
</bean>

<bean id="provider"
    class="com.example.di.decoupled.HellloWorldMessageProvider"/>
```
- @autowired를 사용하여 수정자 의존성 주입
```java
@Service("renderer")
public class StandardOutMessageRenderer implements MessageRenderer {
    @Override
    @Autorwired
    public void setMessageProvider(MessageProvider provider){
        this.messageProvider = provider;
    }
}
```
xml에서 \<context:compoent-scan\> 태그를 선언했기 때문에 스프링은 ApplicationContext를 초기화하는 동안 @Autowired 어노테이션을 발견하고 필요에 따라 의존성을 주입한다. 
#### 생성자 주입
- 외부에서 정의된 메시지 인자 사용하는 클래스 - 전제
```java
public class ConfigurableMessageProvider implements MessageProvider {
    private String message;

    public ConfigurableMessaegProvider(String message){
        this.message = message;
    }
}
```
message값을 전달하지 않으면 위의 인스턴스를 생성할 수없다. 이러한 클래스는 생성자 주입을 하기에 이상적이다.
- Bean 정의
```xml
<bean id="messageProvider"
    class="com.example.di.xml.ConfigurableMessaegProvider">
    <constructor-arg value="Constructor Injection!"/>
</bean>
```
위의 구성 파일에서는 \<property\>를 사용하는 대신 \<constructor-arg\>를 사용했다. 다른 빈은 전달하지 않고 단순 문자열만 전달하기 때문에 ref 대신 value를 사용해서 생성자 인수 값을 지정한다. 둘 이상의 생성자 인수가 있거나 클래스에 둘 이상의 생성자가 있는 경우 index를 사용해서 \<constructor-arg\>에 제공해야한다.

어노테이션을 이용해 생성자 주입을 할 때도 대상 빈의 생성자 메서드에 @Autowired를 사용한다.
```java
@Service("provider")
public class ConfigurableMessageProvider implements MessageProvider {
    private String message;

    @Autowired
    public ConfigurableMessageProvider(@Value("Configurable message")String message){
        this.message = message;
    }
}
```
위의 코드와 같이 @Value를 사용하여 하드코딩하면 메시지를 변경하기할려며 다시 빌드해야한다.

- 메세지를 외부화한 구성파일
```xml
<context:compoent-scan
    base-package="com.example.di.annotated"/>
<bean id="message" class="java.lang.String"
    c:_0="I hope that someone gets my message in a bottle"/>
```
- 생성자 주입을 통해 인자 전달
```java
@Service("provider")
public class ConfigurableMessageProvider implements MessageProvider {
    private String message;

    @Autowired
    public ConfigurableMessageProvider(String message){
        this.message = message;
    }
}
```
message 빈과 해당 빈의 id가 ConfigurableMessageProvider 클래스의 생성자에 지정된 인자의 이름과 동일하게 선언되엇기 때문에 스프링은 @Autowired 어노테이션을 감지하여 값을 생성자 메서드에 주입한다. 

스프링이 생성자를 주입할 때 어떤 생성자를 사용해야할 지 판단 못할 수도있다. 만약 인수의 갯수도 같고 데이터 타입도 같은 두개의 생성자가 있는 경우에 발생한다.
```java
public class ConstructorConfusion {
    private String someValue;
    public ConstructorConfusion(String someValue){
        System.out.println("ContstructionConfusion(String) called");
        this.someValue = someValue;
    }
    
    public ConstructorConfusion(int someValue){
        System.ot.println("ConstructionConfusion(int) called");
        this.someValue = "Integer: " + Integer.toString(someValue);
    }

    public static void main(String... args){
        GenerickXmlApplicationContext ctx = new GenericXmlApplicationContext();
        ctx.load("classpath:spring/app-context-xml.xml");
        ctx.refresh();
        ConstructorConfusion cc = (ConstructorConfusion)ctx.getBean("constructorConfusion");
        System.out.println(cc);
        ctx.close();
    }
}
```
- 구성 파일
```xml
<bean id="provider"
    class="com.example.di.xml.ConfigurableMessageProvider"
    c:message="I hope that someone gets my message in a bottle"/>

<bean id="constructorConfusion"
    class="com.example.di.xml.ConstructorConfusion">
    <constructor-arg>
        <value>90</value>
    </constructor-arg>
</bean>
```
위의 구성파일을 사용하여 실행하게 되면 string 생성자만 호출되게 된다. 이 문제를 해결하기 위해서 인수의 데이터 타입을 지정한 xml 파일을 만들어주어야한다.

```xml
<bean id="provider"
    class="com.example.di.xml.ConfigurableMessageProvider"
    c:message="I hope that someone gets my message in a bottle"/>

<bean id="constructorConfusion"
    class="com.example.di.xml.ConstructorConfusion">
    <constructor-arg type="int">
        <value>90</value>
    </constructor-arg>
</bean>
```
\<constructor-arg\>에 type을 명시해주었다. 위의 구성파일을 사용하여 실행하면 int 생성자가 호출되는 것을 확인할 수 있다.

#### 필드 주입(Field Injection)
필드 주입은 생성자나 수정자를 이용하지 않고 필드에 직접 의존성을 주입한다. 의존성이 필요한 객체의 내부에서만 주입된 의존성을 사용핼 때, 필드 주입을 사용하면 개발자가 빈 초기 생성시 의존성 주입에만 사용되는 코드를 장성하지 않아도 되기 때문에 편리하다. 
```java
@Servide("singer")
public class Singer {
    @Autowired
    private Inspiration inspirationBean;
    public void String() {
        System.out.println("..." + insprivationBean.getLyric());
    }
}
```
inpriationBean은 private 이지만 스프링 컨테이너라 리플랙션(reflection)을 이용해 필요한 의존성을 주입하기 때문에 문제 없다. 
- String 타입 멤버 변수를 가지는 빈 클래스
```java
@Getter
@Setter
@Component
public class Inspiration{
    private String lyric = "고양이 키우고 싶다.";

    public Inspiration(@Value("강아지 키우고싶다.")String lyric){
        this.lyric = lyric; 
    }
}
```
스프링 IoC 컨테이너가 생성할 빈의 정의를 찾을 수 있도록 컴포넌트 스캔 설정을 활성화하여야한다.
```xml
<context:component-scan
    base-package="com.example.di.annotated"/>
```
스필ㅇ IoC 커테이너는 inspriation 탕비의 빈 하나를 발견하면 singer 빈의 inspirationBean 멤버페 해당 빈을 주입한다. 그래서 @Value로 지정된 메시지가 출력되게 된다.

하지만 필드 주입은 여러 단점이 있기 때문에 일반적으로 사용을 권장하지 않는다.
- 필드 주입은 의존성을 추가하기에는 편리하지만, 단일 책임 원칙을 위반하지 않도록 주의해야한다. 많은 의존성이 생기면 클래스에 대한 책임이 커지기 때문에 리팩터링 시에 관심사를 분리하기 어렵다. 클래스가 비대해지는 상황은 생성자나 수정자 주입을 사용한 경우 쉽에 알아챌 수 있지만 필드 주입은 알아채기 어렵다.
- 의존성 주입의 책임은 스필ㅇ의 컨테이너에게 있지만 클래스는 public 인터페이스의 메서드나 생성자를 이요해 필요한 의존성 타입을 명확하게 전달해야한다. 피드 주입을 사용하면 어떤 타입의 의존성이 실제로 필요한지 의존성이 필수 여부인지 명확하지 않다.
- 필드 주입은 final 필드에 사용할 수 없다.
- 필드 주입은 의존성을 수동으로 주입해야 하므로 테스트 코드를 작성하기 어렵다.

#### 주입 인자 사용
스프링은 다른 컴포넌트나 단순 값 외에 자바 컬렉션, 외부에 정의도니 프로퍼티, 다른 팩터리의 빈을 주입할 수 있도록 주입 인자(injection parameter)에 많은 옵션을 지원한다.

#### 단순 값 주입
빈에 단순 값을 주입하기 위해서는 \<value\> 태그로 감싸서 구성 태그에서 지정하면 된다. 

- 단순 값 주입을 위한 빈 클래스
```java
@Setter
public class InjectSiple{
    private String name;
    private int age;
    private float heght;
    private boolean programmer;
    private Long ageInSecond;

    public static void mina(String... args){
        GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
        ctx.load("classpath:spring/app-context-simple-xml.xml");
        cts.refresh();

        InjectionSimple simple = (InjectionSimple)ctx.getBean("injectionSimple");
        System.out.println(simple);
        ctx.close();
    }
}
```
- 구성파일로 단순 값 입력
```xml
<bean id="injectSimple"
    class="com.example.di.xml.InjectionSimple"
    p:name="jinho jeong"
    p:age="26"
    p:height="1.92"
    p:programmer="false"
    p:ageInSecond="819936000">
```