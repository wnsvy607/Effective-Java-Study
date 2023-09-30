<h1>15. 클래스와 멤버의 접근 권한을 최소화하라.</h1>
<br>
<br>
<h2>개요</h2><p>잘 설계된 컴포넌트는 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 완벽히 숨겨서 <b>구현과 API</b>를 깔끔히 분리한다. 컴포넌트간 통신은 <b>오직 API를 통해서만 이루어지며</b> 내부 동작 방식에는 전혀 관심이 없다. 이것을 <b>정보 은닉 혹은 캡슐화</b>라고 한다. 이것은 소프트웨어 설계의 근간이 되는 원리라고 한다.
그리고 자바에서 캡슐화를 달성하기 위해 필요한 원칙 중 하나가 바로 <b>클래스와 멤버의 접근 권한 최소화</b>이다.</p>
<br>
<br>
<br>

<h2>정보 은닉의 장점</h2><ul>
<li>시스템 개발 속도를 높인다.&rarr; 여러 컴포넌트를 병렬로 개발할 수 있기 때문이다.컴포넌트간에 설계된 인터페이스를 통해서 통신하기 때문에 사용하는 측과 구현하는 측 모두 인터페이스에 맞춰서 개발하면 된다. 이는 각 모듈의 동시 개발을 가능케한다.(팀 단위 개발에서 효율적)</li>
<li>시스템 관리 비용을 낮춘다.각 컴포넌트를 더 빨리 파악할 수 있기 때문이다.특히, 인터페이스를 이용한다면 각 컴포넌트가 어떤 역할을 갖고 있는 지 파악하기 쉽다.</li>
<li>성능 최적화에 도움을 준다.정보 은닉과 모듈화를 통해서 다른 컴포넌트에 영향을 주지 않고 각 컴포넌트의 장애 지점과 병목 현상을 개선할 수 있기 때문이다.</li>
<li>소프트웨어 재사용성을 높인다.독자적인 컴포넌트라면 여러 모듈에서 재사용할 수 있다.</li>
<li>시스템 개발 난이도를 낮춘다.<code>Divide and Conquer</code> 장황하고 어려운 시스템도 나누어서 개발하면 좀 더 쉽게 접근할 수 있다.</li>
</ul>
<br>
<br>
<br>
<h2>원칙</h2>
<p>기본 원칙</p><blockquote>
<p>모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다. 달리 말하면, 소프트웨어가 올바로 동작하는 한 항상 가장 낮은 접근 수준을 부여해야 한다는 뜻이다.</p>
</blockquote>
<ul>
<li>톱레벨 클래스와 인터페이스의 접근 수준은 <code>package-private</code>과 <code>public</code> 두 가지가 있다.
<ol>
<li><code>public</code><code>public</code>으로 선언하면 외부에서 <b>접근 가능한 API</b>가 되므로 * ** <b>하위 호환성을 유지하려면 영원히 관리</b>해야 한다.*하위 호환성을 지키지 않는다 == API를 바꾸면 해당 API를 사용하는 클라이언트들의 코드를 거기에 맞춰 변형해야 하는 경우**하위 호환성을 지키는 것에는 장단점이 있으므로 하위 호환성을 깨트리는 코드 변경을 금지시킬 필요는 없다.</li>
<li><code>package-private</code>패키지 외부에서 쓰지 않을 클래스나 인터페이스는 <code>package-private</code>으로 선언해야 한다. <code>package-private</code> 선언함으로써 API가 아닌 내부 구현으로 만드는 것이 가능하다.&rarr; 그렇다면 인터페이스를 구현하는 클래스를 외부 패키지에서 public하게 접근 가능해야 하는가? 그것은 내부 구현으로 <code>package-private</code>으로 숨기는 것이 좋다고 한다.하지만, 스프링 개발자로서 <code>package-private</code>으로 숨기면 <code>Bean</code>으로 등록 가능한 지 여부가 마음에 걸렸는데 간단한 실험을 해보니 내부 구현인 클래스를<code>package-private</code> 으로 선언해도<code>@Service</code>, <code>@Component</code> 를 사용한 컴포넌트 스캔이 정상적으로 동작했다. (물론 당연하게도 외부 패키지에서는 내부 구현체만 모를뿐 인터페이스는 반드시 알아야 한다.)</li>
</ol>
</li>
<li>한 클래스에서만 사용하는 <code>package-private</code> 클래스나 인터페이스는 이를 사용하는 클래스 안에  <code>private static</code>으로 중첩시켜보자.<code>private static</code>으로 중첩시킨 내부 클래스(<code>inner class</code>)는 톱레벨 클래스에서만 접근 가능하다. 좀 더 깊은 정보 은닉이 가능해진다. 하지만, 이 보다 중요한 것은 톱레벨 클래스를 <code>public</code>이 아닌 <code>package-private</code>으로 선언하는 것이라고 한다.</li>
</ul>
<p>*왜 그냥 <code>private</code>이 아닌   <code>private static</code>으로 선언하라고 할까?
그냥 <code>private</code>으로 선언한 내부 클래스는 외부 클래스에 대한 참조를 내부적으로 항상 포함하고 있다. 반면, <code>private static</code>으로 선언한 클래스는 외부 클래스에 대한 참조를 전혀 갖고 있지 않다.
즉, <b><code>private</code> 이너 클래스는 톱레벨 클래스의 멤버에 접근할 수 있지만, <code>private static</code>은 불가능</b>하다. 다시 말해서, <b>톱레벨 클래스와 독립적으로 사용하기 위해 <code>private static</code> 클래스를 사용</b>하는 것이다.</p>
<br>
<br>
<br>

<p>멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준은 네 가지다.</p><ul>
<li><code>private</code>: 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.</li>
<li><code>package-private</code>: 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. 접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준이다.(단, 인터페이스의 멤버는 기본적으로 public이 적용된다).</li>
<li><code>protected</code>: <code>package-private</code>의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다(제약이 조금 따른다).</li>
<li><code>public</code>: 모든 곳에서 접근할 수 있다.</li>
</ul>
<blockquote>
<p>클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 <code>private</code>으로 만들자. 그런 다음 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 (<code>private</code> 제한자를 제거해) <code>package-private</code>으로 풀어주자. 권한을 풀어주는 일을 자주하게 된다면 여러분 시스템에서 컴포넌트를 더 분해해야 하는 것은 아닌지 다시 고민해보자.</p>
</blockquote>
<br>
<br>
<br>
<p><code>private</code>과 <code>package-private</code> 은 내부 구현이라고 이해하면 된다. 따라서 외부에서 사용하는 메서드 혹은 *필드만 public으로 선언하면 된다.</p>
<p>*필드는 <code>public static final</code>로 선언되는 상수를 제외하고는 <code>public</code>으로 선언하는 것을 권장하지 않는다. (아이템 16에서 상세히 다룰 것임) 주의할 점은 상수는 배열(혹은 가변 객체)이어서는 안된다. (즉, primitive 타입 혹은 불변 객체여야 한다) 왜냐면 변경이 될 여지가 있기 때문이고 이는 Thread-unsafe하다. 다른 말로 하자면 race condition에 놓인다. (당연히 배열인 상수를 메서드로 제공하는 것도 권장하지 않는다.)</p>
<br>
<br>
<br>
<p><code>public final</code>로 선언한다면 어떨까? 여전히 해당 필드는 외부로 공개된 API이기 때문에 해당 필드를 없애는 방법으로는 리팩토링이 불가능하다.
가끔씩 테스트할 때 <code>package-private</code>으로 필드를 풀어줘야 할 필요가 있는 경우가 있다. 예를 들어서, <code>private</code>한 필드의 값을 검증하고 싶을 때 Getter가 없다면 해당 클래스 내부의 필드에 값을 검증하고 싶어도 접근이 불가능하기 때문에 시도조차 할 수 없다.</p><pre><code>// RestaurantService.class
public class RestaurantService {

	private final GroceryService groceryService;

	public RestaurantService(GroceryService groceryService) {
		this.groceryService = groceryService;
	}
}

// RestaurantServiceTest.class
class RestaurantServiceTest {

	GroceryService groceryService;


	@Test
	void test() throws Exception {
		RestaurantService restaurantService = new RestaurantService(groceryService);
	// groceryService 는 restaurantService 내부의 private field 이다.
	// 따라서 테스트 코드에서 접근이 불가능하다.
		Assertions.assertNotNull(restaurantService);
		Assertions.assertNotNull(restaurantService.groceryService);
	}
}
</code></pre><p>이럴 경우 크게 두 가지 선택지가 있다.</p><ol>
<li>기존에 있던 public Getter로 접근한다. (오직 테스트 코드만을 위해 만들지는 말자)<pre><code>// RestaurantService.class
	public GroceryService getGroceryService() {
		return groceryService;
	}	
</code></pre></li>
<li>필드를 <code>package-private</code>으로 선언해 테스트 코드에서 해당 필드를 접근할 수 있도록한다.테스트 코드를 테스트 대상이 같은 패키지에 두면 <code>package-private</code> 클래스 혹은 멤버에 접근할 수 있게 되기 때문이다.<pre><code>// RestaurantService.class
	// private을 package-private으로 전환
   	final GroceryService groceryService;
</code></pre></li>
<li>(또 다른 방법) Getter를 <code>package-private</code>으로 선언해준다.<pre><code>   // RestaurantService.class
	GroceryService getGroceryService() {
		return groceryService;
	}
</code></pre></li>
</ol>
<p>위의 3가지 방법을 이용하면 테스트 코드에서 내부 구현 멤버에 접근이 가능하다.</p>
<br>
<br>
<br>
<p>사실 상태(필드)를 내부(생성자 or 메서드)에서 직접 검증하는 것이 가장 최선의 방법이다. (이 경우에는 생성자에서 필드가 할당이 되기 때문에 생성자에 불변식을 만들었다.)</p><pre><code>// RestaurantService.class
public RestaurantService(GroceryService groceryService) {
	if(groceryService == null)
		throw new IllegalArgumentException("groceryService must not be null");
	this.groceryService = groceryService;
}
</code></pre>

<br>
<br>
<br>
<p>단지 테스트를 위해서 <code>public</code> 멤버를 만드는 것은 지양하자.</p>

<ul>
<li><code>protected</code>는 공개 범위가 상당히 넓다. <code>public</code> 클래스의 <code>protected</code> 멤버는 공개 API이므로 하위 호환성 유지를 원한다면 영원히 관리해야 한다. 또한 내부 동작 방식을 API 문서에 적어 사용자에게 공개해야할 수도 있다.(아이템 19 - 나중에 알아볼 내용이다.) 따라서 <code>protected</code> 멤버는 적을수록 좋다.</li>
<li><code>Serializable</code></li>
</ul><blockquote>
<p><code>Serializable</code>을 구현한 클래스는 그 필드들(<code>private</code>, <code>package-private</code>)도 의도치 않게 공개 API가 될 수도 있다.</p>
</blockquote>
<p>이펙티브 자바 완벽 공략 1부 - 완벽 공략 13에 있다고 한다. 기억이 잘 안나는데 <code>Serializable</code> 복습이 필요할 것 같다. 요지는 내부 구현 필드더라도 포맷을 변경하면 역직렬화(<code>Deserialize</code>)가 불가능하기 때문에 포맷을 유지해야만 한다는 것이다.</p>

<ul>
<li>다른 주의 점은, 오버라이딩할 때는 접근 범위를 좁힐 수 없다는 것이다. 해당 제약이 있는 이유는 리스코프 치환 원칙을 지키기 위해서이다. 이 제약 때문에 캡슐화가 어려워질 수도 있다.</li>
<li>자바 9부터는 모듈 시스템의 사용으로 모듈이 다르다면 public과 protected 이어도 외부 모듈에서는 사용할 수 없는 암묵적 접근 수준이 생겼다. 하지만 모듈 시스템을 적극 사용하는 예는 JDK뿐이다. 아직은 직접 모듈 시스템을 사용하는 것은 시기상조라고 한다.</li>
</ul>

