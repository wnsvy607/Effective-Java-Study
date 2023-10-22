<h1>20. 추상클래스보다 인터페이스를 우선하라.</h1>
<br>
<br>
<br>
<h2>개요</h2><p>추상클래스 대신 인터페이스를 우선적으로 활용하자. 인터페이스의 장점이 더 많다.</p>
<br>
<br>

<h3>기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.</h3><p><b>2개 이상의 클래스를 동시에 상속받는 것은 불가능</b>하다. 여러 클래스를 상속 받아야 하는 클래스가 이미 추상 클래스를 상속받았다면 이미 둘 간의 계층구조가 생겨 상속에 어려움이 생긴다. 반면 인터페이스는 한 클래스에서 여러번 구현하게 할 수 있고 기존에 어떤 클래스를 상속받고 있는 지에 관계 없이 기능을 추가할 수 있다.</p>
<br>
<h3>인터페이스는 믹스인이 가능하다.</h3><blockquote>
<p>믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 ‘주된 타입’ 외에도 특정 <b>선택적 행위를 제공한다고 선언하는 효과</b>를 준다.</p>
</blockquote>
<p>인터페이스를 통해 어떤 클래스의 갖고 있는 주된 역할외에 제공하고 싶은 <b>부가 기능을 선택 제공</b>할 수 있다.
예를 들어서 <code>Autocloseable</code> , <code>Comparable</code> , <code>Serializable</code>등이 있다. 추상 클래스는 2개 이상 상속이 불가능하기 때문에 믹스인으로 쓸 수 없다.</p>
<br>
<h3>계층구조가 없는 타입 프레임워크를 만들 수 있다.</h3><p>관계(계층구조가)가 명확하지 않은 경우가 있을 수도 있다. 책에서는 <code>Singer</code>와 <code>Songwriter</code>를 예시로 들었다.</p>
<br>

<h3>인터페이스도 디폴트 메서드를 제공할 수 있다.</h3><p>자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있다. 해당 인터페이스를 구현하는 클래스에 메서드를 추가하고 싶은 경우, 보통 public하게 다른 곳에서 참조해서 모두 변경해야 하는 경우가 많기 때문에 추가하게 된다면 해당 인터페이스를 구현하는 클래스들이 모두 깨지게 된다. 이런 경우, 공통적으로 제공하고 싶은 메서드를 디폴트 메서드로 제공하면 된다. 해당 디폴트 메서드를 깨진 클래스들에서 구현하지 않더라도 인터페이스에서 디폴트로 구현체를 넣어주기 때문이다. 또한 <code>stactic</code>메서드도 제공할 수 있다.</p>
<br>
<h3>디폴트 메서드도 만능이 아니다.</h3><p>디폴트 메서드는 필드를 사용할 수 없기 때문에 필드를 사용해야 하는 경우는 디폴트 메서드로 구현이 불가능하다. 골격 클래스와 같이 사용하면 이런 단점을 보완할 수 있다.</p>
<br>

<h3>래퍼 클래스 관용구(아이템 18)와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.</h3><p>아이템 18 참조</p>
<br>
<h3>인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 제공한다.</h3><p>추상 골격 구현 클래스란 추상 클래스로서 인터페이스를 구현하지만 일부 로직은 완전한 구현체를 제공하고 일부 로직은 사이 사이에 구현해야할 틈이 있는 템플릿 역할을 하는 메서드는 구현이 되어있다. 템플릿 중간에는 하위클래스에서 오버라이딩 할 수 있는 부분들을 남겨 놓은 클래스다. 즉, 추상 클래스가 제공하는 기능들을 활용하면서 쉽게 인터페이스를 구현할 수 있게 도와주는 역할을 한다.</p>
<p>대표적인 예시로 <code>AbstractCollection</code>, <code>AbstractSet</code>, <code>AbstractMap</code>, <code>AbstractList</code> 등이 있다.
인터페이스인 <code>List</code>를 구현하기 위해서는 20개가 넘는 메서드를 구현해야 한다. 하지만 <code>AbstractList</code>를 상속한다면 단 2개의 메서드만 구현해주면 된다. 즉, 상속해서 사용하는 프로그래머에게 미리 쓸만한 부분들을 미리 구현해두어 수고를 덜 수 있게 하는 것이다.
하지만, 추상 골격 구현 클래스를 사용할 때는 <b>상속을 이용할 때의 주의할 점들을 반드시 기억하고 적용시켜줘야</b> 한다.(아이템 18, 아이템 19)</p>
<br>
<br>
<h3>골격 구현 작성?</h3><blockquote>
<p>골격 구현 작성은 (조금 지루하지만) 상대적으로 쉽다. 가장 먼저, 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다. 이 기반 메서드들은 골격 구현에서는 추상 메서드가 될 것이다. 그 다음으로, 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다. 단, <code>equals</code>, <code>hashCode</code> 같은 <code>Object</code>의 메서드는 디폴트 메서드로 제공하면 안 된다는 사실을 항상 유념하자. 만약 인터페이스의 메서드 모두가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다. 골격 구현 클래스에는 필요하면 <code>public</code>이 아닌 필드와 메서드를 추가해도 된다.</p>
</blockquote>
<ul>
<li>‘<code>equals</code>, <code>hashCode</code> 같은 <code>Object</code>의 메서드’는 애초에 문법적으로 디폴트 메서드로 선언하지 못하도록 되어있으므로 걱정하지 않아도 된다. (그렇게 설계한 이유는 다음 포스팅에)</li>
</ul>

<br>
<br>
<h3>시뮬레이트한 다중 상속</h3><p>위에서 언급한 추상 골격 구현 클래스를 이용해 이미 다른 클래스를 상속한 클래스에서 마치 여러개를 상속한 것 처럼 쓸 수도 있다. 다음 예시를 보며 이해해 보자.</p><pre><code>//AbstractDog 추상 클래스
public abstract class AbstractDog {
	public abstract String bark();
	public abstract String move();
}
// AbstractDog을 상속한 Dog 클래스
public class Dog extends AbstractDog {
	@Override
	public String bark() {
		return "멍멍";
	}
	@Override
	public String move() {
		return "강아지가 움직인다.";
	}
}
</code></pre>
<p>다음 코드에서 Dog에서 다른 클래스를 확장(상속)하는건 불가능하다. 하지만, 추상 골격 구현 클래스를 이용해 마치 상속한 것 처럼 사용할 수 있다. 아래 <code>AbstractFlyable</code> 클래스의 기능을 추가하기 원한다고 가정하자.</p><pre><code>// Flyable 인터페이스
public interface Flyable {
	String fly();
}
// AbstractFlyable 클래스
public class AbstractFlyable implements Flyable {
	@Override
	public String fly() {
		return "날아라.";
	}
}
//
</code></pre>
<p>절차는 다음과 같다.</p><ol>
<li><code>Flyable</code> 인터페이스를 구현한다.</li>
<li><code>AbstractFlyable</code> 클래스를 상속받는 <code>private</code> 이너 클래스를 만든다.</li>
<li>해당 이너 클래스를 <code>private</code> 멤버 필드로 선언한다.</li>
<li>구현이 필요한 메서드에서 방금 선언한 멤버 필드로 해당 메서드를 호출한다.</li>
</ol>
<p>완성된 코드를 확인해보자.</p><pre><code>public class Dog extends AbstractDog implements Flyable {

	private MyFlyable myFlyable = new MyFlyable();
	@Override
	public String bark() {
		return "멍멍";
	}
	@Override
	public String move() {
		return "강아지가 움직인다.";
	}

	@Override
	public String fly() {
		return myFlyable.fly();
	}

	private class MyFlyable extends AbstractFlyable  {
		// 해당 메서드는 반드시 오버라이딩할 필요는 없다.
		@Override
		public String fly() {
			return "내가 날아요.";
		}
	}
}
</code></pre>
<p>결과적으로 <code>Flyable</code>의 기능을 확장할 수 있게 되었고 마치 2개의 클래스를 상속한 것과 같은 효과를 낼 수 있다.</p>

