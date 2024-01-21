# Item 34 - int 상수 대신 열거 타입을 사용하라

## 개요 
**필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.** 

또한 열거 타입에 정의된 상수 개수가 영원히 고정불변일 필요는 없다.

## 정수 열거 패턴(int enum pattern) 기법의 문제점

정수 열거 패턴이란 다음과 같이 단순히 정수 상수를 한 묶음 선언하는 것을 말한다.
```java
	// 	코드 34-1 정수 열거 패턴 상당히 취약하다!
	public static final int APPLE_FUJI = 0;
	public static final int APPLE_PIPPIN = 1;
	public static final int APPLE_GRANNY_SMITH = 2;

	public static final int ORANGE_NAVEL = 0;
	public static final int ORANGE_TEMPLE = 1;
	public static final int ORANGE_BLOOD = 2;
```

해당 패턴은 많은 문제점이 존재한다.

1. 타입 안전을 보장할 방법이 없다.
2. 표현력이 좋지 않다.(접두어를 써서 이름 충돌을 방지하기 때문 - 자바는 정수 열거 패턴을 위한 namespace를 지원하지 않는다.)
3. 잘못된 인자를 보내도 그대로 사용할 수 있는 여지가 있다.
<br>
<br>

예시 코드)

```java
    // 향긋한 오렌지 향의 사과 소스!
    int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
```


4. 컴파일 시 그 값이 클라이언트 파일에 그대로 새겨지기 때문에 값이 변경된다면 클라이언트 파일도 다시 컴파일해야한다.
5. 문자열로 출력하기 까다롭다.
6. 같은 정수 열거 그룹에 속하는 모든 상수를 순회하는 방법도 마땅치 않다.
7. 상수가 몇개인지도 알 수 없다.

이 외에도 문자열 열거 패턴(string enum pattern)도 있으나
<br> 
오타등을 컴파일러로 확인할 수 없어 자연스럽게 런타임 버그와 문자열 비교에 의한 성능 저하가 발생한다.



## 열거타입이란?
```java
	// 코드 34-2 가장 단순한 열거 타입
	public enum APPLE{FUJI, PIPPIN, GRANNY_SMITH}
	public enum ORANGE{NAVEL, TEMPLE, BLOOD}
```
자바의 열거 타입은 완전한 형태의 클래스라서 단순한 정숫값일 뿐인 다른 언어의 열거 타입보다 훨씬 강력하다.
<br>
<br>

### 메커니즘
- 자바는 열거 타입의 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다. 
- 열거타입은 **밖에서 접근할 수 있는 생성자를 제공하지 않으므로** 사실상 `final`이며 클라이언트는 인스턴스를 직접 생성하거나 상속할 수 없으므로 **인스턴스가 하나임이 보장**된다. 즉, 열거 타입은 **인스턴스가 통제**되며 **싱글톤**이라고 할 수 있다.



## 장점
### 컴파일 타임 타입 안정성을 제공한다.
단순한 정수 열거 패턴과는 다르게 `APPLE`과 `ORANGE`는 다른 타입이기 때문에 `APPLE`열거 타입을 매개변수로 받는 메서드에 `ORANGE`를 전달한다면 **컴파일 에러가 발생할 것**이다.
<br>
<br>
<br>

### 새로운 상수를 추가하거나 순서를 바꿔도 (클라이언트가) 다시 컴파일 하지 않아도 된다.
열거 타입에서 **공개되는 것은 오직 필드의 이름**뿐이기 때문에 **클라이언트에 값이 각인되어 컴파일되는 것이 아니기 때문**이다.
<br>
<br>
<br>

### 열거 타입에는 임의의 메서드나 필드를 추가할 수 있다.
가장 단순하게는 그저 상수 모음일 뿐인 열거 타입이지만, 실제로는 클래스이므로 고차원의 추상 개념 하나를 완벽히 표현해낼 수도 있다. 

예를 들어서 태양계의 여덟 행성을 열거 타입으로 표현한다고 가정해보자.

```java
// 코드 34-3 데이터와 메서드를 갖는 열거 타입 (211쪽)
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```
- 각 행성은 질량과 반지름 필드를 갖고 있으며 이를 통해 표면중력(`surfaceGravity`)을 계산할 수 있다. 
- 특정 질량(`mass`)이 주어지면 행성 표면에서의 무게도 계산할 수 있는 메서드인 `surfaceWeight`가 추가되어 있다.
- 각 열거 타입 상수 오른쪽 괄호 안 숫자는 생성자에 넘겨지는 매개변수로, 이예에서는 행성의 질량과 반지름을 의미한다.

<br>

위와 같이 열거 타입을 만들 때는 주의할 점이 다음과 같다.
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면된다.
- 열거 타입은 근본적으로 불변이라 모든 필드는 `final`이어야 한다. (아이템 17)
- 필드를 `public`으로 선언해도 되지만, `private`으로 두고 별도의 `public` 접근자 메서드를 두는게 낫다. 아이템(16)
- 기능을 클라이언트에 노출해야할 이유가 없다면 내부구현으로서 `private` 혹은 `package-private`으로 구현하자.(아이템 15)
- 열거 타입이 특정 클래스에서만 사용된다면 멤버 클래스로 사용하고 널리 쓰일 것 같다면 톱 레벨 클래스로 사용하자.(아이템24)
  - EX) `BigDecimal` 의 `java.math.RoundingMode`는 널리 쓰일 것을 감안해 멤버 클래스에서 톱 레벨 클래스로 바뀌었다.

<br><br><br>

### `values` 메서드
열거 타입의 강점 중 하나는 해당 열거 타입이 가진 모든 상수가 담긴 배열을 반환하는 `values` 메서드가 정의되어 있다는 것이다.

```java
// 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력한다. (212쪽)
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
   }
}
```

위와 같이 **`values` 메서드를 통해 `Planet` 열거 타입의 모든 상수를 불러올 수 있다.**

위의 `main` 메서드의 결과는 다음과 같다. (인수로 200을 줬음)

```
MERCURY에서의 무게는 75.581339이다.
VENUS에서의 무게는 181.010201이다.
EARTH에서의 무게는 200.000000이다.
MARS에서의 무게는 75.920799이다.
JUPITER에서의 무게는 505.935888이다.
SATURN에서의 무게는 213.102823이다.
URANUS에서의 무게는 180.971097이다.
NEPTUNE에서의 무게는 227.252704이다.
```

위와 같이 강력한 메서드를 열거 타입은 기본적으로 제공해준다.

<br>
<br>

### 상수가 제거된다면?

상수가 제거되더라도 `values` 메서드는 정상적으로 동작한다. 
만약 클라이언트 코드에서 제거된 상수를 참조하고 있더라도 클라이언트 코드를 다시 컴파일 할 때 
**디버깅에 유용한 컴파일 오류가 발생**할 것이고 **컴파일 하지 않더라도 런타임에 유용한 예외가 발생**할 것이다.
이런 모든 것들은 정수 상수 패턴을 사용할 때는 기대할 수 없는 것들이다.

<br>
<br>

### 그 외

- 각자의 이름 공간(namespace)가 있어 같은 이름의 상수 사용이 가능하다.
- 열거타입의 `toString` 메서드는 출력하기에 적합한 문자열을 `return`해준다.
- 기본으로 `Object` 메서드들이 높은 품질로 구현되어 있다.
- 기본으로 `Comparable`과 `Serializable`도 구현되어 있다.


<br>
<br>
<br>


## 상수별 메서드 구현
열거 타입에서 상수별로 동작이 달라져야 하는 경우도 있을 것이다.

예를 들어서 연산을 의미하는 `Operation` 열거 타입을 만든다고 해보자.  
`Operation`은 단순한 열거 타입을 넘어 상수에 따라 연산하는 메서드까지 제공이 필요하다고 가정하자.

우선, `switch`문을 이용해 상수의 값에 따라 분기하는 방법이 있다.
```java
enum Operation {
	PLUS, MINUS, TIMES, DIVIDE;

	public double apply(double x, double y) {
		switch (this) {
			case PLUS: return x + y;
			case MINUS: return x - y;
			case TIMES: return x * y;
			case DIVIDE: return x / y;
		}
		throw new AssertionError("알 수 없는 연산: " + this);
	}
}
```
`switch`문을 사용하는 방법은 2가지 단점이 있다.
1. 깨지기 쉬운 코드다. 이유는 새로운 상수가 추가될 경우 `case` 문도 추가해야하기 때문이다. 혹시라도 깜빡한다면 새로 추가한 상수에서 `apply` 메서드를 호출한다면 <br> 런타임에 `"알 수 없는 연산: ${추가된 상수}"`라는 오류 메세지를 내는 `AssertionError`가 발생할 것이다.
2. 마지막 `throw` 문은 기술적으로는 도달할 일이 없지만 기술적으로는 도달할 수 있기 때문에 생략하면 컴파일이 되지 않는다.
<br>
다행히도 이를 우회할 수 있는 방법이 존재하는데 그것이 바로 `상수별 메서드 구현(constant-specific method implementation)`이다.
거두절미하고 코드를 보자.

```java
// 코드 34-6 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입 (215-216쪽)
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }
    public abstract double apply(double x, double y);
}
```


위에서 `apply`와 같은 추상 메서드를 선언하고 상수별로 자신에 맞게 재정의 하는 방법이 있다. 이것을 `상수별 클래스 몸체(constant-specific class body)`
라고 한다. 이를 통해서 새로운 상수를 추가할 때 각 상수에 맞게 반드시 추상 메서드를 재정의해야한다.(그렇지 않으면 컴파일 오류가 발생한다.)

또한 상수별 메서드 구현을 상수별 데이터와 결합할 수도 있는데 위에서 재정의한 `toString`이 바로 그 예시이다. `toString`을 재정의 함으로써 계산식 출력을 다음과 같이 간결하게 할 수 있다.

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for (Operation op : Operation.values())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

실행 결과

```
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```

<br>
<br>

### `fromString`
열거타입에는 상수의 이름을 인수로 받아 그 이름에 해당하는 상수를 반환해주는 `valueOf` 메서드가 자동 생성된다. 이와 비슷하게 `toString`을 재정의한다면 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString` 메서드도 함께 제공하는 것을 고려해보자. 다음 코드는 모든 열거 타입에서 사용할 수 있도록 구현한 `fromString`이다.
```java
    // 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

    // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
```
유의할 점은 다음과 같다.
- 열거 타입의 생성자가 호출된 이후에 정적 필드의 초기화가 이루어지기 때문에 생성자에서는 다른 상수와 더불어 `stringToEnum`과 같은 필드에 접근이 불가능하다. 따라서 위와 같이 정적 필드 선언에서 초기화를 하는 것이 바람직하다.
- `fromStirng` 메서드가 `Optional<T>`을 사용하는 것도 주목하자. 문자열에 해당하는 상수가 없을 수도 있음을 알리고 대처하도록 유도한 것이다. 


<br>
<br>

### 상수별로 코드 공유가 필요하다면?
> 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 

급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해보자. 이 열거 타입은 직원의 시간당 기본 임금과 그날 일한 시간이 주어지면 일당을 계산해주는 메서드를 갖고 있다. 주중에는 오버타임이 발생하면 잔업수당이 주어지고, 주말에는 무조건 잔업수당이 주어진다. `switch` 문을 이용하면 `case` 문을 날짜별로 두어 이 계산을 쉽게 수행할 수 있다.
```java
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
	SATURDAY, SUNDAY;

	private static final int MINS_PER_SHIFT = 8 * 60;
	
	int pay(int minutesWorked, int payRate) {
		int basePay = minutesWorked * payRate;
		
		int overtimePay;
		switch (this) {
			case SATURDAY: case SUNDAY: // 주말
				overtimePay = basePay / 2;
				break;
			default: // 주중
				overtimePay = minutesWorked <= MINS_PER_SHIFT ? 
					0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
		}

		return basePay + overtimePay;
	}
```
간결하지만 관리 관점에서는 위험하다. 왜냐하면 새로운 상수를 추가할 때 반드시 `case` 문에 추가해야 한다. 그렇지 않으면 `공휴일`과 같은 상수가 추가 되더라도 무조건 주중에 받는 급여대로 계산될 것이다.

<br>
이 상황을 타개하기 위한 방법은 (전략 열거 타입 패턴을 제외하고)2가지가 있다.

1. 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣는다.
2. 계산 코드를 평일용과 주말용으로 나눠 각각 도우미 메서드에 작성한 다음 각 상수가 상수별 몸체에서 필요한 도우미 메서드를 적절히 호출하도록 한다.

위 두 방식 모두 코드가 장황해져 가독성이 떨어지고 오류 발생 가능성이 높아진다.



### 해법: 전략 열거 타입 패턴
코드가 복잡해지긴 하지만 가장 깔끔한 방법이 바로 `전략`을 열거 타입으로 생성하는 것이다.
```java
// 코드 34-9 전략 열거 타입 패턴 (218-219쪽)
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
```
전략을 의미하는 `PayType` 이라는 열거 타입을 새로 만들고 기존 상수에 필요한 전략을 열거 타입 상수로 선언해서 가지고 있는 패턴이다. 

이 패턴은 다음과 같은 장점이 있다. 
- 상수간 코드 공유가 가능하다.(중복 제거)
- `switch` 문과 달리 새로운 상수 추가에도 안전하다.

<br>
<br>

### 그럼 `switch` 문은 언제 쓸까?
그럼에도 불구하고 `switch` 문이 좋은 선택지가 될 수도 있다.



#### 기존 열거 타입에 상수별 동작을 혼합해 넣을 때
```java
// 코드 34-10 switch 문을 이용해 원래 열거 타입에 없는 기능을 수행한다. (219쪽)
public class Inverse {
    public static Operation inverse(Operation op) {
        switch(op) {
            case PLUS:   return Operation.MINUS;
            case MINUS:  return Operation.PLUS;
            case TIMES:  return Operation.DIVIDE;
            case DIVIDE: return Operation.TIMES;

            default:  throw new AssertionError("Unknown op: " + op);
        }
    }
}
```
서드파티에서 가져온 `Operation`을 위와 같이 기능을 추가할 수도 있다.

<br>
<br>

이외에 직접 만든 열거 타입이라도 추가할 메서드가 
- 의미상 열거 타입에 속하지 않을 때
- 종종 쓰이지만 열거 타입 안에 포함할만큼 유용하지 않을 때

`switch` 문 적용을 고려할 수 있다.

<br>
<br>

## 결론
> 그래서 열거 타입은 언제써야 할까?

- **필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자**. 태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 타입은 당연히 포함된다. 그리고 **메뉴 아이템, 연산 코드, 명령줄 플래그 등 허용하는 값 모두를 컴파일 타임에 이미 알고 있을 때**도 쓸 수 있다.
- **열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.** 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다.

<br>
<br>

> **핵심 정리**
- **정수 상수보다 열거 타입을 사용**하자.
  - 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.
- 하나의 메서드가 상수별로 다르게 동작해야할 때에는
  - `switch` 문 대신 **상수별 메서드 구현을 사용**하자.
  - 열거 타입 상수 일부가 **같은 동작을 공유**한다면 **전력 열거 타입 패턴을 사용**하자