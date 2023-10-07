# springboot&JPA

#### 연관관계 매핑 분석  
-회원과 주문: 일대다, 다대일의 양방향 관계. 따라서 연관관계의 주인을 정하기 위해 외래 키가 있는 주문을 연관관계의 주인으로 정하는 것이 좋다.  

그러므로 Order.member를 ORDERS.MEMBER_ID 외래키와 매핑  
-주문상품과 상품: 다대일 양방향 관계. 외래키가 주문상품에 있으므로 주문상품이 연관관계의 주인이므로 OrderItem.order를 ORDER_ITEM.ORDER_ID 외래키와 매핑  
*외래 키가 있는 곳을 연관관계의 주인으로 정해라!->유지보수/성능을 위해  
#### 엔티티 설계시 주의점  
1. 엔티티에는 가급적 Setter를 사용하지 말자(변경 포인트가 너무 많아서 유지보수가 힘듦)  
2. 모든 연관관게는 지연로딩으로 설정!  
   즉시로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어려움. 특히 JPQL을 실행할 때    N+1 문제가 자주 발생  
   실무에서 모든 연관관계는 지연로딩(LAZY)으로 설정. 연결된 엔티티를 함께 DB에서 조회해야 하면,    fetch join 또는 엔티티 그래프 기능을 사용  
   @XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이기 때문에 직접 지연로딩으로 설정★
3. 컬렉션은 필드에서 초기화
   null 문제에서 안전. 하이버네이트는 엔티티를 영속화 할 때 컬렉션을 감싸서 하이버네잍가 제공하는 내장 컬렉션으로 변경한다.
   만약 getOrders()처럼 임의의 메서드에서 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에서 문제가 발생할 수 있다.
   
+ 논리명 생성: 명시적으로 컬럼, 테이블명을 직접 적지 않으면 ImplicitNamingStrategy 사용
  spring.jpa.hibernate.naming.implicit-strategy:테이블이나 컬럼명을 명시하지 않으면 논리명 적용
+ 물리명 적용: spring.jpa.hibernate.naming.physical-strategy:모든 논리명에 적용
#### 계층적 구조 사용  
- controller, web: 웹 계층
- service: 비즈니스 로직, 트랜잭션 처리
- repository: JPA를 직접 사용하는 계층, 엔티티 매니저 사용
- domain: 엔티티가 모여 있는 계층. 모든 계층에서 사용
** cascade=CascadeType.ALL: persist할 때에 자동으로 persist(참조관계가 유일할 경우에만 쓸 것. 다양한 참조관계가 있는 경우에는 repository 따로 만들어서 CRUD
  
#### 도메인 모델 패턴  
 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할을 한다. 이처럼 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것  
#### 트랜잭션 스크립트 패턴  
 엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것  
#### 동적 쿼리 -> Querydsl로 처리

#### Spring
⭐**Spring FrameWork**  
자바의 오픈소스 애플리케이션 프레임워크 중 하나입니다.
특정 기술에 종속되지 않고 객체를 관리할 수 있는 프레임워크를 제공하는 것 입니다.
스프링 컨테이너로 자바 객체를 관리하면서 DI와 IoC를 통해 결합도를 낮추게 됩니다.

⭐**DI (Depedency Injection)**  
DI는 의존성 주입을 의미합니다.
생성자 주입, Field 주입, Setter 주입을 통해 객체간의 의존관계를 미리 설정해두면 스프링 컨테이너가 의존관계를 자동으로 연결해줍니다.
이렇게 되면 직접 의존하는 객체를 생성하는 일이 없기 때문에 결합도가 낮아지는 장점이 있습니다.
Spring에서 가장 권장하는 의존성 주입 방법은 생성자를 통한 주입 방법 인데 그 이유는 순환 참조를 방지하고, 불변성을 가지며 테스트에 용이하기 때문입니다.

**생성자 주입을 권장하는 이유**
1. 순환 참조를 방지할 수 있다.
개발을 하다 보면 여러 컴포넌트 간에 의존성이 생깁니다.
예를 들어, A가 B를 참조하고, B가 다시 A를 참조하는 순환 참조되는 코드가 있다고 가정해 봅니다.
@Service
public class AService {
 
    // 순환 참조
    @Autowired
    private BService bService;
 
    public void HelloA() {
        bService.HelloB();
    }
}
@Service
public class BService {
    
    // 순환 참조
    @Autowired
    private AService aService;
 
    public void HelloB() {
        aService.HelloA();
    }
}
필드 주입과 수정자 주입은 빈이 생성된 후에 참조를 하기 때문에 어플리케이션이 아무런 오류 그리고 경고 없이 구동됩니다.
그리고 그것은 실제 코드가 호출될 때까지 문제를 알 수 없다는 것입니다.
반면, 생성자를 통해 주입하고 실행하면 BeanCurrentlyInCreationException이 발생하게 됩니다.
순환 참조 뿐만아니라 더 나아가서 의존 관계에 내용을 외부로 노출 시킴으로써 어플리케이션을 실행하는 시점에서 오류를 체크할 수 있습니다.
2. 불변성(Immutability)
생성자로 의존성을 주입할 때 final로 선언할 수 있고, 이로인해 런타임에서 의존성을 주입받는 객체가 변할 일이 없어지게 됩니다.
하지만 수정자 주입이나 일반 메소드 주입을 이용하게되면 불필요하게 수정의 가능성을 열어두게 되고,
이는 OOP의 5가지 원칙 중 OCP(Open-Closed Principal, 개방-폐쇄의 원칙)를 위반하게 됩니다.
그러므로 생성자 주입을 통해 변경의 가능성을 배제하고 불변성을 보장하는 것이 좋습니다.
또한, 필드 주입 방식은 null이 만들어질 가능성이 있는데, final로 선언한 생성자 주입 방식은 null이 불가능합니다.

@Controller
public class CocoController {
 
    private final CocoService cocoService;
 
    public CocoController(CocoService cocoService) {
        this.cocoService = cocoService;
    }
}
 
3. 테스트에 용이하다.
생성자 주입을 사용하게 되면 테스트 코드를 좀 더 편리하게 작성할 수 있습니다.

public class CocoService {
    private final CocoRepository cocoRepository;
    
    public CocoService(CocoRepository cocoRepository) {
    	this.cocoRepository = cocoRepository;
    }
    
    public void doSomething() {
        // do Something with cocoRepository
    }
}
 
public class CocoServiceTest {
    CocoRepository cocoRepository = new CocoRepository();
    CocoService cocoService = new CocoService(cocoRepository);
    
    @Test
    public void testDoSomething() {
        cocoService.doSomething();
    }
}
독립적으로 인스턴스화가 가능한 POJO(Plain Old Java Object)를 사용하면, DI 컨테이너 없이도 의존성을 주입하여 사용할 수 있습니다.
이를 통해 코드 가독성이 높아지며, 유지보수가 용이하고 테스트의 격리성과 예측 가능성을 높일 수 있다는 장점이 생기게 됩니다.
위와 같은 이유로 필드 주입이나 수정자 주입 보다는 생성자 주입의 사용을 권장합니다.

⭐**IoC (Inversion of Control)**  
IoC는 제어의 역전을 의미합니다.
제어권이 개발자에게 있지 않고, 프레임워크에 있어서 필요에 따라서 사용자의 코드를 호출하게 됩니다.
스프링에서는 인스턴스의 생성부터 소멸까지 인스턴스 생명주기 관리를 개발자가 아닌 컨테이너에서 대신 관리하게 됩니다.  

⭐**스프링 Bean 등록**  
스프링 컨테이너에 등록한 객체들을 '빈'이라고 합니다.  
스프링 컨테이너에 Bean을 등록하는 두 가지 방법은 다음과 같습니다.  
**1. 컴포넌트 스캔과 자동 의존관계 설정**  
클래스 선언부 위에 @Component 어노테이션을 사용하는 것입니다.  
@Controller, @Service, @Repository는 모두 @Component를 포함하고 있으며
해당 어노테이션으로 등록된 클래스들은 스프링 컨테이너에 의해 자동으로 생성되어 스프링 빈으로 등록됩니다.  
**2. 자바 코드로 직접 스프링 빈 등록**  
이번에는 수동으로 스프링 빈을 등록하는 방법에 대해 알아보겠습니다.  
수동으로 스프링 빈을 등록하려면 자바 설정 클래스를 만들어 사용 해야 합니다.
XML로 스프링 빈을 등록하는 방법도 있지만, 최근에는 거의 사용하지 않습니다.
설정 클래스를 만들고  @Configuration 어노테이션을 클래스 선언부 위에 추가하면 됩니다.
그리고 특정 타입을 리턴하는 메소드를 만들고, @Bean 어노테이션을 붙여주면 자동으로 해당 타입의 빈 객체가 생성됩니다. 
MemberRepository는 인터페이스이고, MemoryMemberRepository가 구현체이기 때문에  MemoryMemberRepository를 new 해줍니다.
![image](https://github.com/hyeyeonnnnn/spring2/assets/115079024/c2b1a35e-c973-4247-aabb-b1d14444f000)


@Bean 어노테이션의 주요 내용은 아래와 같습니다.  
@Configuration 설정된 클래스의 메소드에서 사용가능
메소드의 리턴 객체가 스프링 빈 객체임을 선언함
빈의 이름은 기본적으로 메소드의 이름
@Bean(name="name")으로 이름 변경 가능
@Scope를 통해 객체 생성을 조정할 수 있음
@Component 어노테이션을 통해 @Configuration 없이도 빈 객체를 생성할 수도 있음
빈 객체에 init(), destroy() 등 라이프사이클 메소드를 추가한 다음 @Bean에서 지정할 수 있음
어노테이션 하나로 해결되는 1번 방법이 간단하고 많이 사용되고 있지만, 상황에 따라 2번 방법도 사용됩니다.
-  1번방법을 사용해서 개발을 진행하다 MemberRepository를 변경해야 할 상황이 생기면 1번 방법은 일일이 변경해줘야 하지만,

   2번 방법을 사용하면 다른건 건들일 필요 없이 @Configuration에 등록된 @Bean만 수정해주면 되므로, 수정이 용이합니다.

**등록된 스프링 빈을 @Autowired로 사용하기**  
스프링 부트의 경우 @Component, @Service, @Controller, @Repository, @Bean, @Configuration 등으로
빈들을 등록하고 필요한 곳에서 @Autowired를 통해 의존성 주입을 받아 사용하는 것이 일반적입니다.

**스프링 컨테이너의 종류 BeanFactory와 ApplicationContext**
1.  BeanFactory
- BeanFactory 계열의 인터페이스만 구현한 클래스는 단순히 컨테이너에서 객체를 생성하고 DI를 처리하는 기능만 제공한다.  
- Bean을 등록, 생성, 조회, 반환 관리를 한다.  
- 팩토리 디자인 패턴을 구현한 것으로 BeanFactory는 빈을 생성하고 분배하는 책임을 지는 클래스이다.  
- Bean을 조회할 수 있는 getBean() 메소드가 정의되어 있다.  
- 보통은 BeanFactory를 바로 사용하지 않고, 이를 확장한 ApplicationContext를 사용한다.  
2.  ApplicationContext  
- Bean을 등록, 생성, 조회, 반환 관리하는 기능은 BeanFactory와 같다.
- 스프링의 각종 부가 기능을 추가로 제공한다.

**⭐AOP (Aspect Oriented Programming)**  
트랜잭션이나 로깅, 보안과 같이 공통적으로 사용하는 기능들을 분리하여 관리할 수 있는 것을 말합니다.

**⭐⭐⭐Spring Container**  
애플리케이션이 실행되면 비어있는 스프링 컨테이너가 생성되고 스프링 설정 파일이나 어노테이션 기반으로 컨테이너에 스프링 빈이 등록되고 의존관계가 주입됩니다.


**⭐⭐⭐Spring MVC**  
MVC는 Model, View, Controller의 약자이며, 각 계층간의 기능을 구분하는데 중점을 둔 디자인 패턴 입니다.
MVC패턴을 사용하는 이유는 비즈니스 로직과 UI 로직을 분리하여 유지보수를 독립적으로 수행하기 위함 입니다.
Model은 데이터 관리 및 비즈니스 로직을 처리하는 부분 입니다. (DAO, DTO, Service)
View는 비즈니스 로직의 처리 결과갸 유저 인터페이스에 표현되는 구간 입니다.
Controller는 사용자의 요청을 처리하고 Model과 View를 중개하는 역할을 합니다.

**⭐⭐@Transactional의 동작 원리**  
@Transactional을 메서드 또는 클래스에 명시하면, AOP를 통해 Target이 상속하고 있는 인터페이스 혹은 Target 객체를 상속한 Proxy 객체가 생성되며,Proxy 객체의 메서드를 호출하면 Target 메서드 전 후로 트랜잭션 처리를 수행합니다.

**⭐Spring의 스코프 프로토타입 빈**  
프로토타입 빈은 싱글톤빈과는 달리 컨테이너에게 빈을 요청할 때마다 매번 새로운 객체를 생성하여 반환해줍니다.

**⭐POJO**  
스프링에서 생성되어 관리되는 POJO 기반의 객체를 Spring Bean이라고 합니다.
여기서 POJO는 단순 getter, setter만으로 구성되어 있으며 단순히 new를 통해서 생성 가능한 형태를 말합니다. 핵심은 특정 기술에 종속되는 어떤 클래스도 상속하지 않고 있고, 어떠한 인터페이스도 구현하고 있지 않은 자바 클래스입니다.

**⭐서블릿**  
서블릿은 자바를 사용해 웹을 만들기 위해 필요한 기술 입니다.
클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술 입니다.
Spring MVC에서 Controller로 이용되며, 사용자의 요청을 받아 처리한 후에 결과를 반환합니다.

**⭐Spring, SpringBoot 차이**  
가장 큰 차이점은 Auto Configuration의 차이인 것 같습니다. Spring은 프로젝트 초기에 다양한 환경설정을 해야 하지만, Spring Boot는 설정의 많은 부분을 자동화하여 사용자가 편하게 스프링을 활용할 수 있도록 돕습니다. spring boot starter dependency만 추가해주면 설정은 끝나고, 내장된 톰캣을 제공해 서버를 바로 실행할 수 있습니다.


**⭐WAS (Web Application Server)와 WS (Web Server) 차이**  
WAS는 비즈니스 로직을 넣을 수 있고, 대표적으로 Tomcat, PHP, ASP 등이 있습니다.
반대로 WS는 비즈니스 로직을 넣을 수 없으며 Nginx, Apache 등이 있습니다.


**⭐@RequestBody, @RequestParam, @ModelAttribute 차이**  
@RequestBody는 클라이언트가 전송하는 JSON 형태의 HTTP Body내용을 MessageConverter를 통해 Java Object로 변환시켜주는 역할을 합니다.
값을 주입하지 않고도 값을 변환시키므로 (Reflection을 사용해 할당), 변수들의 생성자, getter, setter가 없어도 정상적으로 할당됩니다.
@RequestParam은 1개의 HTTP 요청 파라미터를 받기 위해 사용합니다. @RequestParam은 필수 여부가 true이기 때문에, 기본적으로 반드시 해당 파라미터가 전송되어야 합니다.
전송되지 않으면 400Error를 유발할 수 있고, 반드시 필요한 변수가 아니면 required의 값을 false로 설정해야 합니다.
@ModelAttribute는 HTTP Body 내용과 HTTP 파라미터의 값들을 생성자, Getter, Setter를 통해 주입하기 위해 사용합니다.
값 변환이 아닌 값을 주입시키므로 변수들의 생성자나 getter, setter가 없으면 변수들이 저장되지 않습니다.

**⭐VO, BO, DAO, DTO 차이**  
DAO(Data Access Object)는 DB 데이터에 접근하기 위한 객체를 말합니다. (Repository, Mapper)
BO(Business Object)는 여러 DAO를 활용해 비즈니스 로직을 처리하는 객체를 말합니다. (Service)
DTO(Data Transfer Object) 각 계층간의 데이터 교환을 위한 객체를 말합니다.
VO(Value Object) 실제 데이터만을 저장하는 객체를 말합니다.


**⭐대용량 트래픽에서 장애가 발생 시 대응 방법**  
Scale-Up을 통해 하드웨어 스펙을 향상시키거나, Scale-Out을 통해 서버를 어러대 추가해 시스템을 증가시키는 방법이 있습니다.

**⭐JPA란**  
JPA(Java Persistence API)는 자바 진영의 ORM 기술에 대한 API 표준 명세입니다.
즉 보다 객체와 데이터베이스간의 관계를 편리하게 이어주는 것입니다.
JPA 구현체의 대표적인 것으로 Hibernate라는 ORM프레임워크가 있습니다.

**⭐영속성 컨텍스트란**  
영속성 컨텍스트는 엔티티를 영구 저장하는 환경으로, 애플리케이션과 데이터베이스 사이에서 객체를 보관하는 가상의 환경입니다.
영속성 컨텍스트의 생명주기는 트랜잭션과 동일합니다. 트랜잭션을 종료하면 같이 종료됩니다.

 
**⭐엔티티 생명주기**  

비영속: 영속성 컨텍스트와 전혀 관계가 없는 상태

영속: 영속성 컨텍스트에 저장된 상태

준영속: 영속성 컨텍스트에 저장되었다가 분리된 상태

삭제: 삭제된 상태



**⭐영속성 컨텍스트의 장점**  
1차캐시: 똑같은 걸 두번 조회하는 경우, 처음 조회할 때 해당 데이터를 1차 캐시에 올려 두 번째 조회 시에는 쿼리문을 수행하지않고 캐시에서 가져옴으로서 생산성을 높입니다.
 

동일성 보장: 1차캐시에 이미 있는 엔티티이면 그대로 반환하기때문에 여러 번 조회했을 때 동일성을 보장합니다.


트랜잭션을 지원하는 쓰기 지연: 커밋이 되기 전까지는 쿼리문을 쓰기 지연 저장소에 모아뒀다가 커밋하는 시점에 한번에 flush합니다.

 
변경 감지(Dirty Checking): 영속성 컨텍스트가 관리하는 영속 상태 엔티티를 대상으로, 해당 엔티티가 변경되면 update쿼리를 수행해줍니다.

1차 캐시에 처음 저장할 때 동시에 스냅샷 필드를 저장하는데,그 후 commit이나 flush가 일어날 때 엔티티의 현재 값과 스냅샷을 비교하고 변경 사항이 있으면 update 쿼리를 수행하는 것입니다.

지연로딩: 예를 들어 1:다의 관계가 있을 때, 1만 확인하고 싶을땐 굳이 다의 데이터까지 가져올 필요가 없다. 이럴때 지연로딩으로 설정해두면 다에관해선 가짜 객체인 프록시를 가져옴으로서 해결합니다. 추후 실제로 조회가 필요할때 실제 엔티티를 가져옵니다.  


**⭐ORM**  
ORM은 object relational mapping의 약자로
객체와 관계형 데이터베이스의 데이터를 자동으로 매핑(연결) 해주는 것을 말한다  


#### JAVA

**⭐객체지향이란?**  
프로그래밍에서 필요한 데이터를 추상화시켜 상태와 행위를 가진 객체를 만들고 그 객체들 간의 유기적인 상호작용을 통해 로직을 구성하는 프로그래밍 방법입니다.  
**⭐객체지향 프로그래밍의 장점**  
코드 재사용이 용이 => 남이 만든 클래스를 가져와서 사용가능  
유지보수가 쉬움 => 절차지향 프로그래밍은 일일이 찾아 수정해야하는 반면에 해당하는 부분만 수정하면됨  
대형 프로젝트에 적합 => 클래스 단위로 모듈화 시켜서 개발가능  
**⭐객체 지향적 설계 원칙**  
**SRP(Single Responsibility Principle)** : 단일 책임 원칙 클래스는 단 하나의 책임을 가져야 하며 클래스를 변경하는 이유는 단 하나의 이유이어야 한다.  
**OCP(Open-Closed Principle)** : 개방-폐쇄 원칙 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.  
**LSP(Liskov Substitution Principle)** : 리스코프 치환 원칙 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.  
**ISP(Interface Segregation Principle)** : 인터페이스 분리 원칙 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다.  
**DIP(Dependency Inversion Prinsiple)** : 의존 역전 원칙 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다.   
**⭐객체지향 프로그래밍 키워드**    
**추상화** : 불필요한 정보는 숨기고 중요한 정보만을 표현함으로써 공통의 속성이나 기능을 묶어 이름을 붙이는 것  
**캡슐화** : 기능과 특성의 모음을 "클래스"라는 "캡슐"에 넣어서 분류해서 넣는 이 캡슐화다.  
**상속** :상속은 부모클래스의 속성과 기능을 그대로 이어받아 사용할 수 있게하고 기능의 일부분을 변경해야 할 경우 상속받은 자식클래스에서 해당 기능만 다시 수정(정의)하여 사용할 수 있게 하는 것이다. 다중 상속은 불가하다.  
**다형성** : 하나의 변수명, 함수명 등이 상황에 따라 다른 의미로 해석될 수 있는 것이다.즉, 오버라이딩(Overriding), 오버로딩(Overloading)이 가능하다는 얘기다.  
-오버라이딩 : 부모클래스의 메서드와 같은 이름, 매개변수를 재정의 하는 것.  
-오버로딩 : 같은 이름의 함수를 여러 개 정의하고, 매개변수의 타입과 개수를 다르게 하여 매개변수에 따라 다르게 호출할 수 있게 하는 것.  
**⭐제네릭이 무엇인가요?**  
제네릭(Generic)은 클래스 내부에서 지정하는 것이 아닌 외부에서 사용자에 의해 지정되는 것을 의미한다.  
**⭐컬렉션 클래스에서 제네릭을 사용하는 이유를 설명하세요**    
컬렉션 클래스에 저장되는 인스턴스 타입을 제한하여 런타임에 발생할 수 있는 잠재적인 모든 예외를 컴파일 타임에 잡아 낼 수 있어서 사용합니다.  
**⭐데드락이 무엇이고, 해결방법에 대해 설명해보세요**   
둘 이상의 스레드가 lock을 획득하기 위해 기다리는데, 이 lock을 잡고 있는 스레드도 똑같이 다른 lock을 기다리면서 서로 블락상태에 놓이는 것을 말한다  
우선순위를 선정해 자원을 선점하도록 하는 것과 공유 불가능한 상호 배제 조건을 제거하는 것이 있다.  
**⭐JVM이 하는 역할이 무엇인가요?**  
자바 컴파일러가 .java 파일을 컴파일 하면, .class라는 자바 바이트코드로 변환시켜줍니다. 이때 바이트 코드가 기계어가 아니기 때문에 운영체제에서 바로 실행을 못하는데 이때 운영체제가 이해할 수 있도록 해석해주는 것이 JVM입니다.  
컴파일 -> 바이트 코드 -> 기계어 이런식으로 중간에 바이트 코드 과정이 있기 때문에 속도와 메모리에서 단점이 될 수 있다.  
JVM을 사용하면 운영체제에 상관없이 같은 코드를 사용할 수 있습니다.  


