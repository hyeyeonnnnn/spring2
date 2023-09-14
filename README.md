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
