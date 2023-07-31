# 개요
클라이언트 코드가 어떤 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다. 하지만 이 방식은 단점이 존재하기 때문에 대안으로 정적 팩터리 메서드를 생각해 볼 수 있다.

<br><br><br> 

## 정적 팩터리 메서드?
정적 팩터리 메서드는 간단하게 생성자와 같이 클래스의 인스턴스를 반환하는 역할을 한다.

영어로는 static factory methods

```
public class CoffeeBean {
	
    // 원두 원산지
	private String region;
    // 원두 품종
	private String variety;
    // 원두 로스팅 시각
	private LocalDateTime roastedAt;

	private CoffeeBean(String region, String variety, LocalDateTime roastedAt) {
		this.region = region;
		this.variety = variety;
		this.roastedAt = roastedAt;
	}

	public static CoffeeBean of(String region, String variety, LocalDateTime roastedAt) {
		return new CoffeeBean(region, variety, roastedAt);
	}
}
```
<br>

> 위의 public static으로 정의된 of 메서드가 바로 정적 팩터리 메서드의 전형적인 예시다. 위의 경우에는 생성자를 private으로 설정하여 외부에서 클래스의 인스턴스를 접근할 수 있는 방법을 of로 제한시켜 두었지만 public 생성자와 정적 팩터리 메서드는 함께 사용할 수 있다.

 

## 장점
정적 팩터리 메서드는 public 생성자만을 사용할 경우와 비교해 5가지의 장점을 지니고 있다
<br><br><br>
 

1. 이름을 가질 수 있다
위의 코드를 예로 들어보자. 커피 원두의 원산지와 품종 둘 중 하나는 모르는 경우가 발생할 수 있다. 그럼 이 경우에 public 생성자만으로 이 둘을 구분할 수 있을까?
```
	public CoffeeBean(String variety, LocalDateTime roastedAt) {
		this.variety = variety;
		this.roastedAt = roastedAt;
	}

	public CoffeeBean(String region, LocalDateTime roastedAt) {
		this.region = region;
		this.roastedAt = roastedAt;
	}
```
위 처럼 만드는 코드는 **컴파일 불가능**하며 컴파일시 에러가 발생한다.

왜냐하면 생성자도 메서드의 일종으로서 시그니처를 갖는데 위의 코드는 다른 인스턴스를 만들 것을 요구하지만 메서드의 시그니처가 같기 때문에 **어느 것을 호출할 지 모호한 상황이 발생**하기 때문이다.

<br><br>
반면, 정적 팩터리 메서드는 당연하게도 이름을 가질 수 있어서, **반환될 객체의 특성을 쉽게 묘사할 수 있다. 그리고 위와 같이 시그니처가 겹치는 문제도 이름을 통해 해결**할 수 있다.


```
    public static CoffeeBean of(String region, String variety, LocalDateTime roastedAt) {
        return new CoffeeBean(region, variety, roastedAt);
    }

    public static CoffeeBean unknownRegion(String variety, LocalDateTime roastedAt) {
        return new CoffeeBean(null, variety, roastedAt);
    }

    public static CoffeeBean unknownVariety(String region, LocalDateTime roastedAt) {
        return new CoffeeBean(region, null, roastedAt);
    }
```
<br>
원산지를 모를 경우와 품종을 모를 경우 모두가 구현이 되었고 클라이언트도 확실히 어떤 객체가 반환될지 예상할 수 있다.
<br><br><br>


**2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**
즉, 생성자를 private으로 만들고, 정적 팩토리 메서드 만으로 인스턴스 생성을 제한하여, 인스턴스 생성을 제한할 수 있다는 것이다. 이런 클래스를 인스턴스 통제(instance-controlled) 클래스라고 한다.
<br><br>
이를 통해서 객체 생성비용이 큰 경우 인스턴스를 캐싱해놓는 플라이 웨이트 패턴을 구현할 수도 있고, 싱글톤을 구현하는 방법 중의 하나가 될 수 있다. 또한, 인스턴스화 불가 클래스를 만들 수도 있다.
<br><br><br>
**3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다, 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**
즉, 반환타입을 인터페이스로 설정하여 구체적인 타입을 숨길 수 있으며 인터페이스 사용을 강제할 수 있다. 따라서, 다형성에 따라 클라이언트 코드의 변경 없이 구현체를 변경, 확장이 가능하다. 예를 들어서, List.of 메서드는 클라이언트 코드가 List라는 인터페이스만 알고 있으며 구현체가 무엇인지는 신경쓰지도 신경 쓸 필요도 없다. 또한 매개변수에 따라서 각기 다른 클래스를 반환하도록 할수도 있다.
<br>
<br>
자바 8부터는 인터페이스에 static 메서드 선언이 가능하기 때문에 따로 인스턴스화 불가 동반 클래스를 만들 필요가 적어졌다. 자바 9부터는 private static 메서드도 정의가 가능하기 때문에 더욱 필요가 적어졌지만 여전히 static 필드, 클래스가 필요한 경우에는 인스턴스화 불가 동반 클래스가 필요하다.
<br><br><br>
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
반환될 구체 클래스가 없다면 당연히 메서드 실행시 예외가 발생하겠지만 정적 팩터리 메서드를 작성하는 시점에는 예외가 발생하지 않는다. 이렇게 되면 클라이언트의 코드가 구체 클래스에 의존하지 않는다. 따라서 유연함을 얻게 된다.
<br><br><br><br>



## 단점
 
1. **정적 팩토리 메서드만 제공**하면 **상속을 할 수 없다**. <br>
상위 클래스에서 생성자를 private으로 접근을 제한하고 정적 팩토리 메서드만 사용하면 하위 클래스에서 상위 클래스의 생성자를 호출할 수 없기 때문에 하위 클래스를 만들 수 없다.
<br><br><br>
 

2. **프로그래머가 찾기 어렵다.**<br>
생성자는 의도가 명확하고 포맷이 정해져있어 찾기가 어렵지 않지만, **정적 팩토리 메서드 같은 경우** 자바독에서 다른 메서드들과 같이 표시되어 어떤 메서드가 인스턴스를 반환하는 지 혼란스러울 수 있다. 이 문제는 널리 알려진 네이밍 규약에 따르면 어느정도 완화시킬 수 있다. 개발자가 무슨 역할을 하는 메서드인지 이름만 보고 추측할 수 있도록 도와주는 것이다.
<br><br><br>
 

## 네이밍 규약
from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드<br>
of: 여러 매개 변수를 받아 적합한 타입의 인스턴스를 반환하는 메서드<br>
getInstance: 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.<br>
newInstance: 매번 새로운 인스턴스를 생성해 반환함을 보장한다.<br><br>
등이 존재한다.
<br><br><br>


## 그 외
정적 팩터리 메서드는 **GoF 디자인 패턴에서의 팩토리 메서드 패턴과는 연관이 없는 것**이다.
public 생성자와 정적 팩터리 메서드는 각각 장단점이 있으니 상황에 알맞게 사용하면 된다.