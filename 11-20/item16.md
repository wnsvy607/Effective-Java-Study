# 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.
<br>
<br>


## 개요
`public` 클래스에서는 `public`필드가 아닌 `public` 접근자를 만들어 주는 것이 바람직하다.  `public` 필드를 API로 제공하는 대표적인 예시는 `java.awt.package` 패키지의 `Point`, `Dimension` 클래스를 들 수 있다.

<br>
<br>

### 이유
1. 캡슐화의 이점을 제공하지 못한다. (아이템 15)
2. 필드를 변경하려면 API를 변경해야 한다.
3. 필드에 접근할 때 부수 작업을 할 수 없다. 
   `public` 접근자 메서드를 API로 공개한다면 해당 필드에 접근 하기 전에 불변식을 수행하는 것과 같은 부수작업을 수행할 수 있다.


<br>
<br>


### `package-private` 클래스와 `priavate` 중첩 클래스
책에서는 `package-private` 클래스와 `priavate` 중첩 클래스는 데이터 필드를 노출해도 문제가 없다고 한다. 
 하지만, 백기선님 말로는 `package-private` 클래스더라도 필드를 직접 사용하기보다 접근자 메서드를 제공하는 것이 바람직하다고 한다. 이유는 같은 패키지내의 다른 클래스가 사용하더라도 필드만 제공한다면 부수작업을 수행하기 어렵고 `public` 클래스보다 위험이 덜 하더라도 필드 변경시 곧 바로 클라이언트 측에서 그에 맞춘 변경이 필요하기 때문이다.


<br>
<br>


### `public final`  필드
`public` 필드가 불변이어도 마찬가지다. 여전히 메서드를 사용할 때의 장점인 API 변경 없이도 구현을 변경할 수 있다는 것과 부수작업을 수행할 수 있다는 것을 누리지 못한다.

<br>
<br>


### Dimension의 성능 문제
> 아이템 67에서 설명하듯, 내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘까지도 해결되지 못했다.

Dimension이 `public` 필드를 노출한 것은 설계상의 문제라고 볼 수 있는데, 왜 책에서는 **성능 문제**라고 지칭했을까? 왜냐하면 **설계상의 문제점을 해결하기 위해 쓰이는 방법이 성능 문제를 유발하기 때문이다.** 다음 코드를 보자.
```
public void doSomething() {
	Dimension dimension = new Dimension();

	doSomethingDeeper(dimension);

	Integer width = dimension.width;
	Integer height = dimension.height;
	// ...
}
```
doSomething 메서드는 `Dimesion` 객체를 다른 메서드로 넘기고 있다. 문제는 해당 필드가  직접 변경이 가능한`public`  필드라는 것이다. 이는 `doSomethingDeeper`메서드 내부 구현에 따라 의도치 않은 부작용(Side effect)를 일으켜 doSomething 메서드가 의도대로 동작하지 않게 만든다. 

 이런 부작용을 방지하기 위해  `doSomethingDeeper` 메서드는 다음과 같이 받은 파라미터를 카피해서 사용할 수 있다.
```
public void doSomethingDeeper(Dimension param) {
	Dimension dimension = new Dimension();
	dimension.width = param.width;
	dimension.height = param.height;

	System.out.println("param.width = " + param.width);
	System.out.println("param.height = " + param.height);

	// ...
}
```
위와 같이 받은 매개변수를 복사해서 사용한다면 부작용을 확실히 방지할 수 있을 것이다. 즉, `Dimesion`의  `public` 필드라는 설계적 허점때문에 위와 같은 임시방편이 강제되는 것이다. 문제는 이런 새로운 인스턴스를 생성해내는 코드가 수천, 수만, 수억 번 반복되면 성능상의 문제를 야기할 수밖에 없는 것이다. 이것이 책에서 **성능 문제**라고 지칭한 이유인 것이다.


<br>
<br>


## 정리
`public`이든 `package-private`이든 캡슐화와 접근자 제공 방식의 장점들을 누리기 위해 필드를 직접 사용하는 것 대신 항상 `public ` 메서드 접근자를 제공하여 사용하자.