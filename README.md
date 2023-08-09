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
- 
  

