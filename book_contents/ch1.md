# Chapter 1. 도메인 모델 시작하기

## 1.1 도메인이란?
* 정의 : 소프트웨어로 해결하고자 하는 문제 영역
* 상위 도메인을 여러개의 하위도메인으로 분리(구성)할 수 있다.
* 하위 도메인간의 연동을 통해서 완전한 기능이 제공된다
  * ex) 일반적인 온라인 마켓의 경우 - 회원/혜택/상품/주문/결제/배송/정산/통계
* 일부 도메인은 직접 모든 것을 처리하기보다 외부시스템과의 인터페이스 만을 이용하여 처리를 외주화(?)한다.
  * ex) 배송, 결제
* 도메인의 구성은 업태/업체 규모/비즈니스 대상에 따라 달라질 수 있다.  
  
## 1.2 도메인 전문가와 개발자 간 지식 공유
* 요구사항의 정의는 도메인 전문가의 요청을 개발자가 "올바르게 이해"한 뒤에 논의/역제안 등을 거쳐 확정된다.
* 개발자는 이런 요구사항을 분석하고 설계하여 코드를 작성하며 테스트하고 배포한다.
* 요구사항을 올바르게 이해하지 못하면 더 편한길을 돌아서 가거나, 실제 필요한 것과는 다른 엉뚱한 기능을 만들게 된다. 
* 올바르게 이해하려면 - 중간전달자를 없애고 직접 대화하는 것이 좋다
  * 전달자가 많을 수록 정보왜곡/손실 발생
  * 따라서 직접 대화를(효율적으로) 하기위해 개발자도 어느정도의 도메인지식을 갖춰야 함.
* 도메인전문가(일명 현업)
  * 실제 데이터와 소프트웨어의 흐름을 모르는 경우가 많고 
  * 기존에 사용하던 소프트웨어를 기준으로 생각하는 경우가 많다.
  * 따라서 요구하는 기능으로 실제로 하고자 하는 바를 이해해서 실제로 원하는 바를 쉽게 달성할 수 있도록 해주는 것이 중요

## 1.3 도메인 모델
* 정의 : 도메인 모델은 특정 도메인을 개념적으로 표현한 것(개념 모델)
* 역할 : 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는 데 도움이 된다.(주문 이라고 했을때 모두가 동일한 것을 떠올리도록 기본적인 개념을 일치시켜주는 역할)
* 도메인을 이해하는 데 도움이 된다면 표현 방식이 무엇인지는 중요하지 않다.
  * 도메인 객체모델 : 도메인이 제공하는 기능과 도메인의 주요 데이터 구성을 파악하기 용이
  * 상태 다이어그램 : 도메인의 대표적인 상태가 어떻게 전이 되는지 표현하기에 적합
  * 관계가 중요한 도메인리거나 계산 규칙이 중요하다면 그래프나 수학공식을 사용해서 모델을 만들수도 있다.
* 개념모델에서 바로 코드가 나올수는 없으므로 구현 기술에 맞는 구현 모델이 따로 필요하다.

### 하위 도메인과 모델
* 같은 단어도 도메인에 따라 실질적 의미가 달라질 수 있다. 
* 따라서 여러 하위 도메인을 하나의 다이어그램에 모델링하면 안된다. (하위도메인마다 별로 모델 생성)


## 1.4 도메인 모델 패턴
### 아키텍쳐 구성
* 일반적으로 애플리케이션의 아키텍처는 네 개의 영역으로 구성된다.
1. UI 혹은 표현 계층(Presentation)
   * 사용자의 요청을 처리하고 사용자에게 정보를 보여준다.
2. 응용 계층(Application)
   * 사용자가 요청한 기능을 실행한다. *업무 로직을 직접 구현하지 않*으며 도메인 계층을 조합해서 기능을 실행한다.
3. 도메인
   * 시스템이 제공할 도메인 규칙을 구현한다.
4. 인프라스트럭처
   * 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.

* 앞에서 언급된 개념모델로써의 도메인 모델과 달리 앞으로의 도메인 모델은 아키텍처 상의 도메인 계층을 객체 지향 기법으로 구현하는 패턴을 말한다.
* 도메인 계층은 도메인의 핵심 규칙을 구현한다. 
  * (뒤집어 말하면)도메인의 핵심규칙(비즈니스 로직)은 도메인 계층 안에만 존재해야 한다?
  * 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.

### 주문 도메인의 경우
> * 출고 전에 배송지를 변경할 수 있다.
> * 주문 취소는 배송 전에만 할 수 있다.
* 두가지 규칙을 구현한 코드가 OrderState 안에 위치할 수 있다.
* 배송지 변경 가능여부를 판단하는 로직이 OrderState외 다른 것들도 영향을 미친다면 상위의 Order에서 로직을 구현하는 것이 좋다.
* 중요한 점은 주문과 관련된 중요업무 규칙을 주문도메인 모델인 Order/OrderState에서 구현한다는 점.

## 1.5 도메인 모델 도출
* 도메인을 모델링할 때 기본이 되는 작업은 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것이다.
  * 이 과정은 요구사항 정의/분석에서 출발
  * 요구사항들로부터 점진적으로 도메인 모델을 구성해 나가면서 (최초의 요구사항에서 언급되지 않은 사항들에 대해 혹은 추가적으로 발견된 규칙들에 대해) 도메인 전문가나 다른 개발자와 논의하는 과정에서 공유하기도 한다(*요구사항 정련*)

### 요구사항 정리
> * 차량의 렌트/반납 일시와 지점을 정할 수 있다.
> * 반납 지점은 렌트 지점과 달라질 수 있다.
> * 렌트 지점에 따라 반납지점이 동일해야 하는지 제약이 생길 수 있다.(지점 데이터 도메인)
> * 렌트 지점에 따라 보유하고 있는 차종이 달라질 수 있다.
> * 대리점과의 계약에 따라서 여러가지 요금제가 제공될수 있다.
> * 요금제에는 포함사항/불포함 사항이 지정되어있다.
> * 차종에는 승차 가능인원/연료/트랜스미션/기본장착옵션 등의 데이터 가 포함되어 있다.
> * 기본 장착 옵션 외에 GPS/위성라디오/부스터시트등의 추가옵션이 제공될 수 있다.
> * 현지결제/지금결제등의 결제 모드가 분리되어있다. - 요금제 Dependant
> * 지금 결제의 경우 예약 취소와 지정 렌트 시점에 따라 페널티가 부과될 수 있다.
> * 24시간 렌트를 기준으로 Daily Charge가 부과된다.
> * 예약시 입력된 예약자(주운전자) 정보를 통해 실제 렌탈시 예약자를 확인한다. 

```java
class ReservationInfo {
	PickDropInfo pickDropInfo
	VehicleInfo rentalVehicle;
	DriverInfo driverInfo;
	PriceNOptionInfo priceNoptionInfo
	
	Class PickDropInfo {
		StationInfo rentStation, returnStation;
		DateTime rentDT, returnDT;
		int rentalDays;
	}	
	Class PriceNOptionInfo {
		RateInfo rateInfo;
		Currency baseCurrency; // String
		DailyCharge dailyCharge;
		List<AdditionalOptionInfo> additionalOptions;
		
		Class DailyCharge {
			Money base;	// float?Decimal?
			Money option; // float
		}
	}
}
```

* 시작부터 너무 큰 도메인을 잡았다.... 나중에 천천히 분리하자...

### 문서화
* 문서화를 하는 주된 이유는 구체화된 코드보다 상위의 추상화된 문서를 참조하여 전체적인 그림을 그려나 갈 수 있도록하기 위해서임.
* 도메인 내의 실제 움직임은 구체화된 코드를 보면서 이해하게 되므로 코드 자체도 문서화의 대상이 된다.
  * 코드에 도메인 지식이 묻어자니 않으면 코드의 동작은 해석가능해도 도메인관점에서의 비즈니스 로직을 이해하기 어려워진다. (1.7 도메인 언어와 유비쿼터스 언어 와도 연관이 있어보임)


## 1.6 엔티티와 밸류
* 모델의 분류 : 엔티티, 밸류

### 엔티티
* 엔티티는 "고유한", "불변의" 식별자를 가진다. (가장 큰 특징)
  * 따라서 두 엔티티 객체의 식별자가 같으면 두 엔티티는 같다고 판단 가능
* 식별자 생성 방법은
  * 특정규칙에 따라 생성 (202206191839XXXX - DateTIme+Serial)
  * UUID 등의 고유식별자 생성기 사용
  * 값을 직접 입력
  * AutoIncremented Value 사용 - DB에 들어가기 전까지 식별자가 생성되지 않으므로 별?루)
* EX) ReservationInfo, VehicleInfo, StationInfo, RateInfo?, AdditionalOptionInfo?
  
### 밸류
* 밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다. (?)
* 밸류 타입은 꼭 두 개 이상의 데이터를 가져야 하는 것은 아니다. 의미를 명확하게 표현하기 위해 밸류 타입을 사용하는 경우도 있다.
* EX) DriverInfo, PickDropInfo, PriceNOptionInfo, RateInfo, DailyCharge

* 밸류타입에서 기본형이 아닌 Money 타입 등을 만들어서 코드의 의미를 명확화 하는것이 좋다.
* 새로 만든 타입에서는 해당 타입(만)을 위한 기능을 추가해서 사용할 수 있다.
* 밸류 객체의 데이터를 변경할 때는 기존 데이터를 변경하는 것보다 변경한 데이터를 갖는 새로운 밸류 객체를 생성하는 방식이 좋다. (불변성)
* 밸류 타입을 불변으로 구현하는 이유는 참조 투명성과 스레드에 안전한 코드를 작성하기 위함이다.
* 밸류 타입 클래스는 equals()와 hashCode()를 구현해주는 것이 좋다.

### 엔티티 식별자와 밸류 타입
* 단순히 리터럴 값이 아닌, 밸류 타입을 이용하여 해당 값에 의미를 부여하면 가독성이 늘어난다. (String id; -> OrderNo id;)

### 도메인 모델에 set 메서드 넣지 않기
* setter는 도메인의 핵심 개념이 드러나지 않는다.
* 도메인 객체를 생성할 때 온전하지 않은 상태가 될 수 있다.
* 도메인 규칙의 의도가 명확히 드러나도록 메서드의 이름을 지정하고 추가로 필요한 동작들을 함꼐 처리하도록 한다.
  * 해당 메서드의 내부에서는 private으로 선언된 set메서드를 사용 -> ..그냥 변수를 수정하면 안되나? 는 클래스일수도 있지....

## 1.7 도메인 용어와 유비쿼터스 언어
* 코드에서 step1 step2 step3 step4, 15 45 55 99 등의 custom된(?) 용어를 사용하여 상태를 표현하게 되면 해당 상태등을 수정하는 코드도 그 경향을 따라갈 소지가 크다.
* 전문가, 관계자, 개발자가 도메인과 관련된 공통의 언어를 만들고 이를 대화, 문서, 도메인 모델, 코드, 테스트 등 모든 곳에서 같은 용어를 사용한다.(유비쿼터스 언어)
* 소통 과정에서 발생하는 용어의 모호함을 줄일 수 있고 개발자는 도메인과 코드 사이에서 불필요한 해석을 줄일 수 있다.
  * 따라서 도메인규칙이 코드에 그대로 드러날 수 있도록 용어를 지정하는 것이 중요하다. ( beforeStep4 -> beforePayment )
* 도메인에 대한 이해가 깊어지면서 새로운 용어를 찾아내고 공유하여 기존 산출물에도 반영하는 작업이 중요하다. (코드의 현행화? 이것도 기술부채라고 할수 있나?)
* 도메인 용어에 알맞은 단어를 찾는 시간을 아끼워하지 말자.
