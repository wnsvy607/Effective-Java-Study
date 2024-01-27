# Item38 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 요약
열거타입을 확장하고 싶을 때, 열거 타입이 인터페이스를 구현하도록 만들면 된다!

```java
// 코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다. (232쪽)
public interface Operation {
    double apply(double x, double y);
}

// 코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다. - 기본 구현 (233쪽)
public enum BasicOperation implements Operation {
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

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

위와 같이 구성을 한다면 `Operation`을 구현한 또 다른 열거 타입을 정의해 기본 타입인 `BasicOperation`을 대체할 수 있다.

```java
// 코드 38-2 확장 가능 열거 타입 (233-235쪽)
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

이렇게 추가를 한다면 아이템 34의 상수별 메서드 구현과는 달리 추상 메서드를 선언하지 않아도 된다 또한 `Operation` 인터페이스를 사용하도록 작성되어 있기만 하면 열거 타입을 사용가능하다.


<br>
<br>

또한 개별 인스턴스 수준에서뿐 아니라 타입 수준에서도, 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다.
```java
   // 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예 (234쪽)
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```
위 코드는 `ExtendedOperation`의 모든 원소를 테스트하는 코드다. 여기서 `<T extends Enum<T> & Operation>` 구문이 조금 복잡한데, Class 객체가 열거 타입인 동시에 `Operation`의 하위 타입이어야 한다는 뜻이다.

<br>
<br>

두 번째 대안은 Class 객체 대신 한정적 와일드카드 타입(아이템 31)인 `Collection<? extends Operation>을 넘기는 방법이다.
```java
// 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예 (235쪽)
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet,
                            double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```
이 코드는 여러 구현 타입의 연산을 조합해 호출할 수 있는 유연함을 가지게 된다. 하지만, 특정 연산에서는 `EnumSet`과 `EnumMap`을 사용하지 못한다. 인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 사소한 문제가 있다. 바로 열거 타입끼리 구현을 상속할 수 없다는 점이다.
아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다.

공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없애보자.