# Item 32 - 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 개요
**가변인수**와 **제네릭**을 함께 사용하면 힙 오염이 발생할 여지가 있으므로 신중히 사용해야만 한다.(2가지 조건을 준수하자.)


## 힙오염이 발생 가능한 이유?
[[가변인수]]를 사용하게되면 원래는 불가능한 제네릭으로 배열을 만드는 것이 가능해진다. 책에서 힙오염이 발생한다고 지목한 예제코드를 보자.
```
// 코드 32-1 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다! (191-192쪽)  
static void dangerous(List<String>... stringLists) {  
    List<Integer> intList = List.of(42);  
    Object[] objects = stringLists;  
    objects[0] = intList; // 힙 오염 발생  
    String s = stringLists[0].get(0); // ClassCastException  
}
```
`stringList` 는 List의 배열이고 Object의 배열로 쓸 수 있다.(배열은 공변이기 때문에)
여기서 `objects`를 Object로서만 활용을 한다면 문제가 없겠지만 `objects` 0번째 원소로 `List<Integer>`를 삽입하게 된다면 [[힙오염]]이 발생한다. `List<String>`타입을 담고 있어야할 `stringList` 배열이 `List<Integer>` 타입인 원소를 담고 있게 되면서 
`String s = stringList[0].get(0);` 부분에서 문제가 발생한다. `String` 타입의 변수로 할당하기 위해서 컴파일러가 자동으로 형변환을 수행하는데 첫번째 원소는 `List<Integer>`가 담겨 있으므로 `ClassCastException`이 발생하는 것이다.

해당 코드의 바이트 코드를 보면 자동으로 형변환(타입 캐스팅)을 수행하는 것을 알 수 있다.

```java
static varargs dangerous([Ljava/util/List;)V
	
	...
   
   L3
    LINENUMBER 12 L3
    ALOAD 0
    ICONST_0
    AALOAD
    ICONST_0
    INVOKEINTERFACE java/util/List.get (I)Ljava/lang/Object; (itf)
    CHECKCAST java/lang/String
    ASTORE 3
    
    ...
    
```
`CHECKCAST java/lang/String` 부분이 바로 자동으로 추가된 형변환이다.

이런 힙오염은 제네릭의 설계 의도인 타입 안정성을 해치게 되며 최악의 오류인 런타임에 오류가 발생하도록 만든다.

## 제네릭과 가변인자를 동시에 안전하게 쓰는 방법
안전하지 않은 케이스는 크게 2가지가 있다.
첫 번째 케이스는 앞서 봤듯 제네릭 가변인수 배열에 값을 저장하는 것이다.
두 번째 케이스는 제네릭 가변인수 배열의 참조를 외부로 노출하는 행위이다. 다음 코드를 보자.
```java
// 코드 32-2 자신의 제네릭 매개변수 배열의 참조를 노출한다. - 안전하지 않다! (193쪽)  
static <T> T[] toArray(T... args) {  
    return args;  
}

static <T> T[] pickTwo(T a, T b, T c) {  
    switch(ThreadLocalRandom.current().nextInt(3)) {  
        case 0: return toArray(a, b);  
        case 1: return toArray(a, c);  
        case 2: return toArray(b, c);  
    }  
    throw new AssertionError(); // 도달할 수 없다.  
}  
  
public static void main(String[] args) { // (194쪽)  
    String[] attributes = pickTwo("좋은", "빠른", "저렴한");  
    System.out.println(Arrays.toString(attributes));  
}
```
`toArray`메서드가 반환하는 제네릭 가변인수 배열은 런타임시 `Object[]` 배열 타입이다. 하지만, `String[]` 타입으로 할당하려고 하니 컴파일러가 자동으로 형변환을 실시하지만 `Object[]`에서 `String[]`으로는 형변환이 불가능하다.(하위타입이 아니므로)
그래서 안전하게 쓰는 방법은 

따라서 우리는 제네릭과 가변인자를 동시에 **안전하게 쓰려면** 다음 **2가지 조건을 준수하면 된다.**
1. **varargs 매개변수 배열에 아무것도 저장하지 않는다.**
2. **그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.**

**추가로 메서드가 안전함이 확인되었다면 해당 메서드에 `@SafeVarargs`(가변인자 관련 경고만 숨겨주는)어노테이션을 선언해 경고를 숨겨주자.**

안전하게 사용한 예시코드
```java
// 코드 32-3 제네릭 varargs 매개변수를 안전하게 사용하는 메서드 (195쪽)
@SafeVarargs  
static <T> List<T> flatten(List<? extends T>... lists) {  
    List<T> result = new ArrayList<>();  
    for (List<? extends T> list : lists)  
        result.addAll(list);  
    return result;  
}
```



## 깔끔한 대안 - Item 28
Item 28에서 배웠듯이 가변인수 배열을 쓰는 대신에 다음과 같이 `List`를 사용하면 클라이언트 코드가 약간 지저분 해지고 성능이 조금 느려지지만 안전하고 깔끔하게 사용할 수 있다. 게다가 `@SafeVarargs`나 관련 경고가 뜨지 않기 때문에 메서드를 사용하는 쪽에서도 고민할 필요가 없다.
```java
// 코드 32-4 제네릭 varargs 매개변수를 List로 대체한 예 - 타입 안전하다. (195-196쪽)
static <T> List<T> flatten(List<List<? extends T>> lists) {  
    List<T> result = new ArrayList<>();  
    for (List<? extends T> list : lists)  
        result.addAll(list);  
    return result;  
}
```



