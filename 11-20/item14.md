<h1>14. Comparable을 구현할지 고려하라</h1>
<br>
<br>
<br>
<h2>개요</h2><p><code>Object</code>에 포함되는 인터페이스는 아니지만, 널리 사용될 수 있는 인터페이스이다.</p><ul>
<li><code>Object.equals</code>에 더해서 순서까지 비교할 수 있으며 Generic을 지원한다.</li>
</ul>
<br>
<br>
<br>

<h2><code>CompareTo</code>규약</h2><ol>
<li>반사성</li>
<li>대칭성</li>
<li>추이성</li>
<li>일관성</li>
</ol>

<h3>권장</h3><p><code>compareTo</code>가 0을 반환한다면 <code>equals</code>도 true를 반환하도록 구현하는 것이 좋다.
<code>BigDecimal</code> 클래스의 경우 <code>equals</code>와 <code>compareTo</code>의 결과가 다를 수도 있도록 구현되었다.</p>
<pre><code>BigDecimal oneZero = new BigDecimal("1.0");
BigDecimal oneZeroZero = new BigDecimal("1.00");
System.out.println(oneZero.compareTo(oneZeroZero)); // Tree, TreeMap
System.out.println(oneZero.equals(oneZeroZero)); // 순서가 없는 콜렉션
</code></pre>
<p>결과(출력)</p><pre><code>0
false
</code></pre>

<br>
<br>
<br>

<h2>구현방법</h2><p><code>Comparable&lt;T&gt;</code>를 구현하게 되면 T 타입과 비교할 수 있는 <code>compareTo(T t)</code> 메서드를 오버라이딩해야한다.</p>
<ol>
<li>기본타입의 경우 박싱타입의 <code>compare</code> 메서드를 호출해서 구현해주자.</li>
</ol>

<br>
<br>
<br>

<h2>유의점</h2><ul>
<li>상속을 할 경우 <code>equals</code>의 경우처럼 규약을 지키는 것과 다형성을 사용하는 것 중 양자택일을 해야한다.따라서 이런 경우에는 컴포지션을 쓰는 것을 책에서 권장한다.</li>
<li>자바 8부터는 <code>Comparator</code> 인터페이스의 static 메서드를 활용해 <code>Comparator</code> 인스턴스의 생성이 가능하다. 이를 통해 가독성이 좋은 <code>compareTo</code>를 만들 수 있다.</li>
</ul><pre><code>// 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
private static final Comparator&lt;PhoneNumber&gt; COMPARATOR =
        comparingInt((PhoneNumber pn) -&gt; pn.areaCode)
                .thenComparingInt(pn -&gt; pn.getPrefix())
                .thenComparingInt(pn -&gt; pn.lineNum);

@Override
public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
</code></pre><p>위 코드는 <b>Comparator 를 static import</b> 했다는 것에 주의하자.
성능이 10% 정도 느리다고 하지만, <code>Comparator</code>가 엄청 자주 쓰이는 것이 아니라면 크게 개의치 않아도 된다.</p>
<br>
<br>
<br>
<h2>그 외</h2>
<blockquote>
<p><code>Comparable</code>은 타입을 인수로 받는 제네릭 인터페이스이므로 <code>compareTo</code> 메서드의 인수 타입은 컴파일타임에 정해진다. 입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻이다. - 이펙티브 자바 3판 90p</p>
</blockquote>
<p><code>Comparable</code>이 제네릭 인터페이스이므로 매개 변수가 옳은 지를 컴파일 타임에 체크가 가능하다는 장점이 있다는 뜻이다.</p>


<blockquote>
<p>자바의 타입 추론 능력이 이 상황에서 타입을 알아낼 만큼 강력하지 않기 때문에 프로그램이 컴파일되도록 우리가 도와준 것이다. - 이펙티브 자바 3판 92p</p>
</blockquote>
<pre><code>// 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
private static final Comparator&lt;PhoneNumber&gt; COMPARATOR =
        comparingInt((PhoneNumber pn) -&gt; pn.areaCode)
                .thenComparingInt(pn -&gt; pn.prefix)
                .thenComparingInt(pn -&gt; pn.lineNum);
</code></pre><p>위 코드에서 함께 언급된 내용으로 <code>comparingInt</code>에서는 타입을 명시적으로 선언해야 했지만, 그 다음부터인<code>thenComparingInt</code>에서는 타입추론이 잘 작동한다.</p><ul>
<li>타입 추론은 <code>var</code>로 변수선언을 할 때도 쓰인다.</li>
<li><code>compareTo</code>에서 수를 비교할 경우 단순히 두 수를 더하거나 뺀 값으로 반환한다면 정수 오버플로(또는 언더플로)를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있다. 따라서, 각 기본타입의 래퍼 클래스 (이를테면 Integer, String 등)가 제공하는 정적 메서드 <code>compare</code>를 이용하거나 비교자 생성 메서드를 사용하자.</li>
<li>부동소수점 계산의 경우 부동소수점 타입(float, double)의 실제 표현이 지수부와 가수부로 나뉘는데, 가수부가 2진수로 표현되므로 정확한 표현이 어렵다. 특히, 계산을 하게되면 정보의 손실이 발생하기 때문에 정확한 계산이 안될 수 있다.</li>
<li>따라서, 소수점 아래를 내부적으로 Int로 만들어서 계산하는 <code>BigDecimal</code></li>
<li>다음과 같이 직접 비교할 경우 다음과</li>
</ul><pre><code>@Override
public int compareTo(RestaurantService restaurantService) {
	if(this.num == restaurantService.num)
		return 0;
	if(this.num &gt; restaurantService.num)
		return 1;
	return -1;
}

</code></pre>