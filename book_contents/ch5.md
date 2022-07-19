# Chapter 5. 스프링 데이터 JPA를 이용한 조회 기능

## 5.1 시작에 앞서
* CQRS : Command /Query Responsibility Segregation 명령/조회 책임 분리
* 도메인 모델은 주로 명령모델로 사용된다.
* 정렬/페이징/검색 조건 지정 같은 기능은 조회기능에 사용된다.
* 이 장에서는 도메인 모델의 Repository 와 데이터 접근에 사용하는 DAO를 혼용하여 사용한다.

## 5.2 검색을 위한 스펙
* 검색 조건이 고정적이고 단순하면 특정 조건으로 조회하는 기능을 만들면 된다.
* 목록 조회같은 기능은 다양한 조건의 조합이 필요할 때가 있다.
	* 필요한 조합마다 find메서드 정의하기 :	좋은 방법이 아니다.
	* 스펙(Specification)을 사용해서 애그리거트가 특정 조건을 충족하는지 검사할 수 있다.
```JAVA
public interface Speficiation<T> 
	// agg : 검사 대상이 되는 객체
	//리포지터리에 사용시 agg는 애그리거트 루트
	//DAO에 사용시 agg는 검색결과로 리턴할 데이터 객체
	public boolean isSatisfiedBy(T agg);
}
/////////////////////////////////////
public class OrderSpec implements Specification<Order> {
	private String orderId;
	public OrderSpec(String ordererId) {
		this.ordererId = ordererId;
	}
	public boolean isSatisfiedBy(Order agg) {
		return agg.getOrdererId().getMemberId().getId().equals(ordererId);
	}
}
```
* 모든 애그리거트가 메모리에 올라와있지 않기 때문에 단순하게 아래 처럼 구현하지는 않는다.
```JAVA
public class MemberOrderRepository implements OrderRepository {
    
    public List<Order> findAll(Specification<Order> spec) {
        List<Order> allOrders = findAll();
        return allOrders.stream().filter(order -> spec.isSatisfiedBy(order)).toList();
    }
}
/////////////////////////////////////
//검색 조건을 표현하는 스펙을 생성
Specification<Order> ordererSpec = new OrdererSpec("madvirus");

//리포지터리에 전달
List<Order> orders = orderRepository.findAll(ordererSpec);
```

## 5.3 스프링 데이터 JPA를 이용한 스펙 구현
* 정적 메타 모델을 @StaticMetaModel 애너테이션을 사용해 지정할 수 있다.
	* 문자열로 지정할 수도 있지만 오타가능성과 자동완성기능을 사용할 수 없다.
	* 하이버네이트가 정적 메타모델을 생성하는 도구를 제공하니 활용하자.

* JPA의 Specification인터페이스를 사용해서 Predicate를 생성하여 검색조건을 사용할 수도 있다
	* 아래처럼 별도 클래스에 스펙 생성 기능을 모아도 된다.(함수형 인터페이스 이므로 람다식을 사용하여 객체를 사용할 수도 있다.
```JAVA
public class OrderSpecs {
	public static Specification<Order> orderer(String ordererId) {
		return (root, cb) -> cb.equal(
			root.get(Order_.orderer).get(Orderer_.memberId).get(MemberId_.id), ordererId);
	}

	public static Specification<Order> between(Date from, Date to) {
		return (root, cb) -> cb.between(root.get(Order_.orderDate), from, to);
	}
}
```


## 5.4 리포지터리/DAO에서 스펙 사용하기
* findAll 메서드는 스펙 인터페이스를 파라미터로 받아 스펙에 만족하는 엔티티를 찾아준다.

## 5.5 스펙 조합
* 스펙인터페이스에서 and/or/not메서드를 메서드체이닝을 이용하여 사용할 수 있다.
	* where메서드를 사용해서 nullable 스펙을 편하게 사용할 수 있다.

## 5.6 정렬 지정하기
* **findBy**XXX**OrderBy**YYY**Desc**ZZZ**Asc**
* 너무 길어질 때는 검색 조건/스펙 뒤에 Sort를 붙여서 처리할 수 있다.
	* Sort도 and/or로 메서드체이닝을 사용해서 연결 할 수 있다.
```JAVA
List<OrderSummary> findByOrdererId(String ordererId, Sort sort)
List<OrderSummary> findAll(Specification<OrderSummary> spec, Sort sort);
/////////////////////////////////////////////////////////////////////////////
Sort sort = Sort.by("number").ascending().and(Sort.by("orderDate").descending();
```

## 5.7 페이징 처리하기
* Pageable 인터페이스를 구현한 PageRequest 클래스를 사용해서 페이징 처리를 간단하게 구현 할 수 있다.
	* 리턴타입을 Page로 할 경우, 목록 조회와 함께 COUNT쿼리도 실행하여 전체개수/페이지 개수 등을 함꼐 제공한다.
		* 페이징이 필요없다면 List를 리턴받도록 해서 불필요한 COUNT쿼리를 실행하지 않도록 한다.
		* findAll에 Pageable을 사용하면 무조건 COUNT쿼리를 실행하므로 주의
* findFirstN/findTopN형식의 메서드를 사용하여 처음 N개를 조회할 수도 있다.

## 5.8 스펙 조합을 위한 스펙 빌더 클래스
* 여러개의 스펙을 조합 해야 할 경우 SpecBuilder를 사용하여 작성하면 매우 편리하고 Readable한 코드를 작성할 수 있다.
	* 백오피스등에서 여러 조건으로 검색조건이 들어올 가능성이 있을 경우 유용하게 사용가능할 것으로 보인다.
	* builder()->(and/ifHasText/ifTrue)->toSpec() 을 체이닝 해서 사용

## 5.9 동적 인스턴스 생성
* 쿼리 결과에서 임의의 객체를 동적으로 생성할수 있는 기능을 제공한다.(select 뒤에 new키워드를 사용하여 생성자에 인자로 전달할 값을 지정
	> select new com.myshop.order.query.dto.OrderView( o.number, o.state, m.name, m.id, p.name ) from ~~~~ 
    
	* 조회 전용모델을 통해서 편하게 JPQL을 사용해 객체기준으로 쿼리를 작성하고 연관관계와 즉시/지연로딩에 대한 고민없이 데이터를 조회할 수 있다.
	
## 5.10 하이버네이트 @Subselect 사용
* @Immutable, @Subselect, @Synchronize를 사용하여 쿼리결과를 @Entity로 매핑할 수 있다.
	* DBMS가 View를 사용하는 것처럼 쿼리결과를 매핑할 테이블 처럼 사용한다.
	* View를 수정할수 없듯이 매핑된 엔티티도 수정할수 없으므로 @Immutable을 사용한다.
	* @Synchronize를 사용하여 엔티티를 로딩하기 전에 지정한 테이블에 관련된 변경이 발생했을 경우 flush하여 영속화 한뒤에 조회하도록 할 수 있다.
	* @Subselect의 값으로 지정된 쿼리가 from절의 서브쿼리 형태로 사용되므로 주의를 기울여야 한다
		* 서브쿼리가 싫으면 네이티브/마이바티스등의 매퍼를 사용해서 조회한다.
