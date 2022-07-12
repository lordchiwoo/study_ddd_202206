# Chapter 4. 리포지터리와 모델 구현

## 4.1 JPA를 이용한 리포지터리 구현

### 4.1.1 모듈 위치
* 리포지터리 인터페이스는 애그리거트와 같이 **도메인** 영역에 속하고 / 리포지터리를 구현한 클래스는 **인프라스트럭쳐** 영역에 속한다.
* 팀 표준에 따라 domain.impl 처럼 위치를 바꿀수도 있긴 하나 타협안일 뿐이지 좋은 설계원칙은 아니다.

### 4.1.2 리포지터리 기본 기능 구현
* 기본기능 : 조회/저장
* 인터페이스는 애그리거트 루트를 기준으로 작성.(데이터 일관성)
* 조회 기능 : 결과로 null을 사용하고 싶지 않으면 Optional을 활용한다.
* 조회/저장의 구현 : JPA의 EntityManager를 이용해서 구현한다. (find / persist)
* 저장 시점
		* 애그리거트 수정 결과를 저장소에 반영하는 메서드를 추가할 필요는 없다.
		* Transaction범위에서 변경된 Data는 JPA가 자동으로 DB에 반영해준다.
* 조회
	* 네이밍
		* findBy프로퍼티이름(프로퍼티 값) 형식이 가장 널리 사용된다.
		* 결과가 한개 이상이 리턴될수 있으면 List<>를 사용한다.
	* ID외의 다른 조건으로 조회시 JPA의 Criteria나 JPQL을 사용할 수 있다.
* 삭제 : 삭제 기능이 필요한 경우 삭제할 애그리거트 객체를 파라미터로 전달받는다.
	> 삭제 요구 사항이 있어도 실제로 삭제하기보다는 삭제플래그를 통해 데이터를 노출할지 결정하는 방식으로 구현하는 경우도 있다.

## 4.2 스프링 데이터 JPA를 이용한 리포지터리 구현
* 스프링 데이터 JPA를 사용하면 지정한 규칙에 맞게 인터페이스를 정의했을 때 구현객체를 자동으로 만들어서 스프링빈으로 등록해줄 수 있다.
* 직접 구현하지 않아도 되기 때문에 쉽게 정의 가능
* 자동 등록 규칙
> * **리포지터리 인터페이스에서** org.springframework.data.repository.Repository<T,ID> 인터페이스 상속
> * T는 엔티티 타입 / ID는 식별자 타입을 지정
* 메서드 규칙(Order Repository 기준)
	1. 조회
		* Order findById(OrderNo id)
		* Optional<Order> findById(OrderNo id)
		* List<Order> findByOrderer(Orderer orderer)
		* List<Order> findByOrderMemberId(MemberId memberId)
	2. 저장
		* Order save(Order entity)
		* void save(Order entity)
	3. 삭제
		* void delete(Order entity)
		* void deleteById(OrderNo id)
* 추가 규칙은 5장에서 추가로 알아보자


## 4.3 매핑 구현

### 4.3.1 엔티티와 기본 매핑 구현
* 기본 규칙
	* 엔티티
		* 애그리거트 루트는 엔티티 이므로 @Entity로 매핑 설정한다.
	* 밸류
		* 밸류는 @Embeddable / 밸류 타입 프로퍼티는 @Embedded로 매핑 설정
	* @Embeddable에 설정한 칼럼 이름과 실제 테이블 칼럼 이름이 다른경우
		* @AttributeOverrides 애너테이션 사용
	* 밸류 내부의 칼럼"들"의 이름도 변경할 수 있다.
			* @AttributeOverrides({ @AttributeOverride(name,column), ... @AttributeOverride(name,column)  })
				

### 4.3.2 기본 생성자
* 엔티티와 밸류는 생성자 주입방식을 사용한다.
* 불변인 경우 set메서드를 제공하지 않으며 기본생성자를 추가할 필요가 없다.
* BUT!! JPA에서 @Entity, @Embeddable로 클래스를 매필하려면 기본 생성자를 제공해야 한다.
	* protected 클래스명(){} //JPA를 적용하기 위한 기본생성자의 형태
	* 다른 코드에서 사용하지 못하도록 protected를 사용!
	
### 4.3.3 필드 접근 방식 사용
* 매핑 처리의 방식 : 
	* 필드 : @Access(AccessType.Field)
	* 메서드 : @Access(AccessType.PROPERTY)
* 메서드 방식의 경우 프로퍼티를 위한 get/set메서드를 구현해야 함
	* 도메인의 의도가 사라지고 객체가 아닌 데이터기반으로 엔티티를 구현할 가능성이 높아진다
	* 특히 set의 경우 내부 데이터를 외부에서 변경하는 수단이 되므로 캡슐화를 깨는 원인이 될 수 있다.
	* -> 의도가 잘 드러나는 기능을 제공하자
		* ex) setState() -> cancel()
		* ex2) setShippingInfo() -> changeShippingInfo()
	* 밸류타입을 불변으로 구현하려면 set메서드 자체가 필요없는데 JPA의 구현방식때문에 public set메서드를 추가하는 것도 좋지 않다.
		* 기능중심으로 엔티티 구현을 유도하기 위해 프로퍼티 방식이 아닌 필드 방식으로 선택해서 불필요한 get/set 메서드를 구현하지 않도록 한다.
* 하이버네이트는 명시적 접근방식 지정이 없는경우 @Id, @EmbeddidId가 필드에 위치했는지 get메서드에 위치했는지에 따라 접근방식을 선택한다.
		
### 4.3.4 AttributeConverter를 이용한 밸류 매핑 처리
* 밸류 타입의 프로퍼티를 하나의 칼럼에 매핑하는 경우
	* ex) length.value, length.unit을 합쳐 '1000 mm' 같은 형식으로 저장
	* @Embeddable만으로는 처리 불가능하므로 AttrivuteConverter를 사용한다.
	```JAVA
	//X는 밸류 타입, Y는 DB 컬럼
public interface AttributeConverter<X,Y> {
    public Y convertToDatabaseColumn (X attribute);
    public X convertToEntityAttribute (Y dbData);
}
///////////////////////////////////////////////////
// 1. 구현시 @Converter애너테이션을 사용한다
// 2. autoApply속성은 true일 경우 모든 Money 프로퍼티에 대해 자동으로 적용된다.
// 3. false로 지정(default)된 경우 프로퍼티 별로 컨버터를 직접 지정한다.
@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter<Money, Integer> {

	@Override
	public Integer convertToDatabaseColumn(Money money) {
		if(money == null) return null;
		else return money.getValue();
	}

	@Override
	public Money convertToEntityAttribute(Integer value) {
		if(value == null) return null;
		else return new Money(value);
	}
}
///////////
public class Order {

	@Column(name = "total_amounts")
	@Convert(converter = MoneyConverter.class)
	private Money totalAmounts;
	...
}
	```
	
### 4.3.5 밸류 컬렉션 : 별도 테이블 매핑
* 다른 테이블의 키를 참조하는 테이블의 데이터를 List로 가져오는 경우
	* List에는 인덱스 값이 필요하므로 종속된 테이블에도 인덱스값을 저장하는 칼럼이 존재한다
		* Map은?? 인덱스 칼럼이 없으면 오류가 나나?
* @ElementCollection / @CollectionTable을 함께 사용한다.
```JAVA
@Entity
@Table(name = "purchase_order")
public class Order {
	...
	@ElementCollection
	//밸류를 저장할 테이블 지정 / 속성은 name(테이블 명), joinColumns(외부키로 사용할 칼럼 - 두개 이상인 경우, @JoinColumn의 배열을 이용)
	@CollectionTable(name = "order_line", joinColumns = @JoinColumn(name = "order_number"))
	//List가 가질 인덱스값의 칼럼을 지정
	@orderColumn(name = "line_idx") 
	private list<OrderLine> orderLines;
}

@Embeddable
public class OrderLine {
	@Embedded
	private ProductId productId;
	
	//List가 인덱스를 가지므로 line_idx 프로퍼티가 존재하지 않는다.
	
	...
}
```

### 4.3.6 밸류 컬렉션 : 한 개 컬럼 매핑
* 밸류 컬렉션을 하나의 컬럼에 저장하는 경우 AttributeConverter를 사용한다.
	* ex) 도메인모델에서는 Set으로 이메일을 보관 / DB에는 한 컬럼에 CSV로 저장

```JAVA
//밸류 컬렉션을 표현하는 새로운 밸류타입을 추가해줘야 한다.
public class EmailSet {
	private Set<Email> emails = new HashSet<>();

	private EmailSet() {}
	private EmailSet(Set<Email> emails) {
		this.emails.addAll(emails);
	}

	public Set<Email> getEmails() {
		return Collections.unmodifiableSet(emails);
	}
}

///////////////////////////////////////////////////
//밸류 컬렉션을 위한 타입을 추가했다면 AttributeConverter를 구현한다.
@Converter
public class EmailSetConveter implements AttributeConveter<EmailSet, String> {
	@Override
	public String convertToDatabaseColumn(EmailSet attribute) {
		if(attribute == null) return null;
		return attribute.getEmails().stream()
						.map(Email::toString)
						.collect(Collectors.joining(","));
	}
	@Override
	public EmailSet convertToEntityAttribute(String dbData) {
		if(dbData == null) return null;
		String[] emails = dbData.split(",");
		Set<Email> emailSet = Arrays.stream(emails)
						.map(value ->  new Email(value))
						.collect(toSet());
		return new EmailSet(emailSet);
	}
}

///////////////////////////////////////////////////
//이제 남은 것은 EmailSet 타입의 프로퍼티가 Converter로 EmailSetConverter를 사용하도록 지정하는 것이다.
	@Column(name = "emails")
	@Convert(converter = EmailSetConverter.class)
	private EmailSet emailSet;

```

### 4.3.7 밸류를 이용한 ID 매핑
* 식별자 의미를 부각하기 위해 식별자 자체를 밸류타입으로 만드는 경우 @EmbeddedId 애너테이션을 사용한다
* 장점 : 식별자에 기능을 추가할 수 있다
	* ex) 1세대/2세대 시스템의 주문번호 구분하기 - is2ndGeneration()
* 유의 사항
	* JPA의 식별자 타입은 Serializable이어야 하므로 해당 인터페이스를 상속받는다.
	* JPA의 엔티티 비교를 위해 equals, hashcode()를 구현해줘야 한다.
	
### 4.3.8 별도 테이블에 저장하는 밸류 매핑 **(컬렉션이 아님)**
* 루트엔티티를 제외한 대부분의 요소는 밸류이다. 
* 루트외의 다른엔티티가 있다면 진짜 엔티티인지 의심해봐야함.
* 진짜 엔티티라면 다른 애그리커드는 아닌지 확인해야 한다.
	* 독자적인 라이프 사이클을 갖는가?
	* 생성/변경 시점과 주체가 다른가?
	* 서로의 변경이 서로에게 영향을 주는가?
	* 고유식별자를 갖는가? - 모든 PK가 고유식별자 인것은 아니다.
	* ex) 상품 - 상품 리뷰

* 밸류 매핑 테이블 지정을 위해 @SecondaryTable을 클래스에 사용
* 컬럼이름이 다른경우 위에서 언급되니 @AttributeOverrides를 사용 
* (같은 이름을 갖도록 통일하는 방향으로 가면 어떨까?) -> 테이블을 지정해야 하므로 안 됨..ㅜ

```JAVA
@Entity
@Table(name = "article") 
// @SecondaryTable을 이용하면 아래 코드를 실행할 때 두 테이블을 조인해서 데이터를 조회한다.
// 속성은 name(테이블 명), pkJoinColumns(밸류테이블을 엔티티 테이블로 조인할때 사용한 칼럼 - 두개 이상인 경우, @PrimaryKeyJoinColumn 배열을 이용) 
@SecondaryTable(
	name = "article_content",
	pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public class Article {
	@Id
	private Long id;
	...
	// 조인되는 테이블을 지정해주기 위해 Override사용
	@AttributeOverrides({
		@AttributeOverride(name = "content",
			column = @Column(table = "article_content")),
		@AttributeOverride(name = "contentType",
			column = @Column(table = "artible_content"))
	})
	private ArticleContent content;
	...
}
///////////////////////////////////////////////////

// @SecondaryTable로 매핑된 artible_content 테이블을 조인해서 가져온다.
Article article = entityManager.find(Article.class, 1L);
///////////////////////////////////////////////////

```
* 게시글 목록 화면의 경우 article테이블의 데이터(title)만 필요하지 article_content 테이블 데이터는 필요하지 않다.
* @SecondaryTable 사용시 article_content를 조인해서 읽어오므로 오버헤드가 발생한다.
* 이 문제 해결을 위해 ArticleCOntent를 엔티티로 매핑하고 지연로딩으로 설정할수도 있으나 밸류인 모델을 엔티티로 만드므로 좋지 않다.
	* 조회 전용 기능을 구현하자(5장에서 계속), CQRS도 알아보자(11장에서 계속)

### 4.3.9 밸류 컬렉션을 @Entity로 매핑하기
* 라고는 했지만 구현 기술의 한계나 팀표준때문에 밸류를 @Entity로 사용해야 할 때도 있다.
	* @Embeddable 타입은 클래스 상속 매핑이 지원되지 않는다. 
	* @Entity를 사용해야 함 -> 식별자 매핑을 위한 필드와 타입식별(discriminator) 칼럼(구현 클래스 구분)이 추가되어야 한다
	* 그리고 .... 그리고 ... 그리고 .............
	* **개복잡해지니까 상속을 포기하고 if-else로 가라**
	
### 4.3.10 참조와 조인 테이블을 이용한 단방향 M-N 매핑
* 애그리거트간의 집합 연관은 성능상 이유로 피해야 한다.(3장)
* 그럼에도 불구하고 요구사항 구현에 유리하다면 ID참조를 이용한 단방향 집합연관을 적용해 볼 수 있다.
```JAVA
@Entity
@Table(name = "product")
public class Product {
	@EmbeddedId
	private ProductId id;

	@ElementCollection
	@CollectionTable(name ="product_category",
		joinColumns = @JoinColumn(name = "product_id"))
	private Set<CategoryId> categoryIds;
	...
}
```

* Product -> Category 단방향 M-N 연관 / ID참조방식을 사용
* 밸류 컬렉션 매칭과 동일한 방식으로 설정
* 차이점은 Set의 값으로 밸류가 아닌 연관 식별자를 사용
* @ElementCollection을 사용하므로 Product삭제시 매핑에 사용된 조인테이블의 데이터도 함꼐 삭제된다.
* 애그리거트 직접 참조시 영속성 전파/로딩전략을 고민해야함.(ID참조하면 좋은 점)

## 4.4 애그리거트 로딩 전략
* 애그리거트 루트 로딩시 모든 객체가 완전한 상태여야 한다.
* 즉시로딩을 통해 가능하지만 컬렉션같은 경우 CartesianJoin을 사용하여 쿼리결과에 중복이 발생한다.
* 하이버네이트가 알아서 알맞게 제거하면서 메모리 관리를 하지만 애그리거트가 커지면 문제가 될수 있다.
	* image 2 / option 2 인 경우 4
	* image 20 / option 15인 경우 300...
	* -> 성능 문제가 발생할 수 있다.
* 애그리거트가 완전해야 하는 이유
	* 표현영역에서 애그리거트의 상태정보를 보여주기 위해 -> 별도의 조회전용 기능과 모델을 사용하자
	* 상태 변경 기능 실행시 애그리거트가 완전해야함.
		* JPA는 트랜잭션 범위 내에서 지연로딩을 허용하므로 실제 변경 시점에 필요한 구성요소만 로딩해도 괜찮다.
		* + 일반적으로 상태변경보다 조회 빈도가 훨씬 높다. -> 상태 변경을 위한 지연로딩에 발생하는 추가쿼리로 인한 속도 저하는 보통 문제되지 않는다.
		* + 지연로딩은 동작방식이 항상 동일하므로 경우의 수를 따질 필요가 없는 장점이 있다.
		* - 즉시 로딩보다는 쿼리 실행횟수가 많아질 가능성이 더 높다
		* 무조건은 없고 애그리거트에 맞게 즉시로딩과 지연로딩을 선택하자

## 4.5 애그리거트의 영속성 전파
* 애그리거트 루트의 조회 뿐만 아니라 저장/삭제할 때도 전체 애그리거트를 하나로 처리해야 한다.
	* 저장 메서드 : 루트만 저장하면 안되고 애그리거트에 속한 모든 객체를 저장해야 한다
	* 삭제 메서드 : 루트만 삭제 하면 안되고 애그리거트에 속한 모든 객체를 삭제해야 한다
* @Embeddable 매핑은 함께 저장되고 삭제되므로 cascade속성을 설정할 필요가 없다
* @Entity 타입에 대한 매칭은 cascade 속성을 사용해서 저장과 삭제시에 함께 처리되도록 설정해야한다.
	* @OneToOne, @OneToMany는 cascade속성의 기본값이 없으므로 cascade속성을 설정해주자
```
@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE}, orphanRemoval = true)
```

## 4.6 식별자 생성 기능
* 생성 방식
	* 사용자가 직접 생성
		* ex) 이메일 주소
		* 별다른 처리가 필요없음.(중복검사?)
	* 도메인 로직으로 생성
		* ex) 고객ID와 타임스탬프로 주문번호 생성
		* 규칙을 도메인 서비스에 구현
		* 응용서비스는 도메인 서비스를 이용해서 식별자를 생성
		* 혹은 리포지터리 인터페이스에 식별자 생성 메서드를 추가하고 리포지터리 구현클래스에서 알맞게 구현
	* DB를 이용한 일련번호 사용
		* 식별자 매핑에서 @GeneratedValue(strategy = GenerationType.IDENTITY)
		* 이경우 도메인 객체를 리포지터리에 저장할떄 식별자가 생성된다 
			* 도메인 객체를 생성하는 시점에는 식별자를 알수 없고 저장한 뒤에 식별자를 구할 수 있다.
				* JPA릐 식별자 생성 기능을 사용하는 경우에도 저장 시점에 식별자를 생성함.
			* 참조 : https://gmlwjd9405.github.io/2019/08/12/primary-key-mapping.html

## 4.7 도메인 구현과 DIP
* 4장에서 구현한 리포지터리는 DIP원칙을 어기고 있다. - 도메인 모델이 JPA에 의존하고 있다
* DIP를 구현을 할 수는 있다. 
	* 도메인 계층의 리포지터리 인터페이스를 리포지터리에 구현
	* 도메인 계층의 Article 클래스에서 JPA애너테이션을 삭제하고 인프라에 JPA연동 클래스를 추가
* 그래서 얻는게 뭐냐?
	* 리포지터리와 도메인 모델의 구현기술은 거의 바뀌지 않는다.
	* 바뀌게 될 때 고민을 하는게 낫지 않겠냐.....
	
* 단위테스트
	* JPA에 맞춰 도메인 모델을 구현해야 할때도 있지만 이런 상황은 드물고 리포지터리도 마찬가지다.
* DIP를 완벽하게 지키면 좋겠지만 개발 편의성과 실용성을 가져가면서 구조적인 유연함은 어느정도 유지했다.
* 복잡도를 높이지 않으면서 기술에 따른 구현 제약이 낮다면 합리적인 선택이라고 생각한다.
* **논의 필요**
  
