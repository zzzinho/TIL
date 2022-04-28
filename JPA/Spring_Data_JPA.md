# Spring Data JPA
## 명세(Specification)
명세를 사용해서도 데이터베이스를 조회할 수 있다.  다양한 검색 조건을 조립해서 새로운 검색 조건을 만들 수 있다.
```java
public interface OrderRepository extends JpaRepsitory<Order, Long>, JpaSpecificationExecutor<Order> {

}
```
```java
public interface JpaSpecificationExecutor<T> {
    T findOne(Specification<T> spec);
    List<T> findAll(Specification<T> spec);
    Page<T> findAll(Specification<T> spec, Pageable pageable);
    List<T> findAll(Specification<T> spec, Sort sort);
    long count(Specification<T> spec);
}
```
and(), or(), not()을 사용해서 명세를 조합할 수 있다.
```java
import static org.springframework.data.jpa.domain.Specification.*;
import static jpabook.jpashop.domain.spec.OrderSpec.*; 

public List<Order> findOrders(String name) {
    List<Order> result = orderRepository.findAll(
        where(memberName(name)).and(isOrderStatus())
    );
    return result;
}
```
- 명세 정의 코드
```java
import org.springframework.data.jpa.domain.Specifitaion;
import org.springframework.util.StringUtils;
import javax.persistence.criteria.*;

public class OrderSpec {
    public static Specification<Order> memberName(final String memberName){
        return new Specification<Order>(){
            public Predicate toPredicat(Root<Order> root, CriteriaQuery<?> query, CriteriaBuilder builder) {
                if(StringUtils.isEmpty(memberName)) return null;
                // Order와 Member를 조인
                Join<Order, Member> m = root.join("member", JoinType.INNER);
                // 조인했을 때 멤버의 이름이 memberName과 같은지
                return builder.equal(m.get("name"), memberName);
            }
        };
    }
    public static Specification<Order> isOrderStatus() {
        return new Specification<Order>() {
            public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> query, CriteriaBuilder builder){
                // Order의 status가 OrderStatus.ORDER 인지 검사
                return builder.equal(root.get("status", OrderStatus.ORDER))
            }
        }
    }
}
```
## 사용자 정의 레포지토리
보통 스프링 데이터 JPA를 사용하면 인터페이스만 정의하고 구현체는 만들지 않는다. 하지만 커스텀 레포지토리가 필요한 경우도 있다.
### 예시
```java
public interface MemberRepositoryCustom {
    public List<Member> findMemberCustom();
}
```
인터페이스 이름 + Impl 로 지으면 스프링 데이터 JPA가 사동으로 사용자 정의 구현 클레스로 인식한다.
```java
public class MemeberRepositoryCustomImpl implements MemberRepositoryCustom {
    @Override
    public List<Member> findMemberCustom(){
        /...
    }
}
```
Jpa 레포지토리와 커스텀 레포지토리를 extends 해줌으로 둘 다 사용 가능하다.
```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRespositoryCustom {

}
```

## JPA & QueryDSL
- org.springframework.data.querydsl.QueryDslPredicateExecutor
- org.springfraemwork.data.querydsl.QuerydslRepositroySupport
### QueryDslPredicateExecutor
```java
public interface ItemRepository extends JpaRepository<Item, Long>, QueryDslPredicateExecutor<Item> {

}
```
ItemRepository에서 QueryDSL 사용 가능
```java
QItem item = QItem.item;
Iterable<Item> result = itemReposiroty.findAll(
    item.name.contains("장난감").and(item.price.between(100, 200))
);
```
Paging과 Sort도 사용할 수 있다.

### QueryDslRepositorySupport
```java
public interface CustomOrderRespository {
    public List<Order> search(OrderSearch orderSearch);
}
```
```java
public class OrderRepositoryImpl extends QueryDslRepositorySupport implements CustomOrderRepository {

    public OrderRepository() {
        super(Order.class);
    }

    @Override
    public List<Order> search(OrderSearch orderSearch) {
        QOrder order = QOrder.order;
        QMember member = QMember.member;

        JPQLQuery query = from(order);

        if(StringUtils.hasText(orderSearch.getMemberName())){
            query.leftJoin(order.name, member)
                 .where(member.name.contians(orderSearch.getMemberName()));
        }

        if(orderSearch.getOrderStatus() != null){
            query.where(order.status.eq(orderSearch.getOrderStatus()));
        }

        return query.list(order);
    }
}
```
이런 방식으로 레포지토리를 확장 할 수 있다.