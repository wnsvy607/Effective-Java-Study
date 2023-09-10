
<h2>3장</h2><blockquote>
<p><code>Object</code>는 객체를 만들 수 있는 구체 클래스지만 기본적으로는 상속해서 사용하도록 설계되었다. Object에서 <code>final</code>이 아닌 메서드(<code>equals</code>, <code>hashCode</code>, <code>toString</code>, <code>clone</code>, <code>finalize</code>)는 모두 재정의(overriding)를 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.</p>
</blockquote>
<p>일반 규약에 맞게 해당 메소드를 overriding 해야 일반 규약을 활용하는 클래스들(<code>HashMap</code>, <code>HashSet</code>)이 오동작하지 않게 된다. <code>finalize</code>는 이전 장에서 다루었으므로 더이상 언급하지 않는다.</p>
<br>
<br>
<br>

<h2>개요</h2><p><code>equals</code> 메서드는 overriding하기 쉬워 보이지만 곳곳에 함정이 있으므로 <b>필요하지 않은 경우 overriding하지 않는 것이 최선</b>이다. 그냥 두게되면 해당 인스턴스는 오직 자기 자신과만 같게 된다. 그럼 <code>equals</code>를overriding할 필요가 없는 경우는 어떤 경우가 있을까?</p>
<ul>
<li>각 인스턴스가 본질적으로 고유할 때EX) 싱글톤 객체, <code>Enum</code></li>
<li>논리적인 동치성을 검사할 필요가 없는 경우EX) 문자열</li>
<li>상위 클래스에서 재정의한 equals가 하위 클래스에도 적절하다.EX) <code>AbstractList</code> &larr; <code>List</code>, <code>AbstractSet</code> &larr; <code>Set</code>,</li>
<li>클래스가 <code>private</code>이거나 <code>package-private</code>이고 <code>equals</code> 메서드를 호출할 일이 없다.<code>public</code>이면 <code>equals</code>가 호출되지 않음을 보장할 수 없기 때문이다.</li>
</ul>
<p>반대로 위 경우를 제외하고 객체 식별성(object identify; 두 객체가 물리적으로 같은가)이 아니라 <b>논리적 동치성을 비교해야 하며</b> 상위 클래스에서 <b>논리적 동치성을 비교하도록 <code>equals</code>가 재정의 되지 않았을</b> 때 <b>equals</b>를 overriding 해주어야 한다.</p>

<br>
<br>
<br>


<h2><code>equals</code> 일반 규약</h2><p><code>equals</code> 메서드를 재정의할 때는 <code>Object</code> 명세에 적힌 다음과 같은</p><ol>
<li>반사성(reflexivity)<code>A.equals(A) == true</code></li>
<li>대칭성(symmetry)<code>A.equals(B) == B.equals(A)</code></li>
<li>추이성(transitivity)<code>A.equals(B) && B.equals(C)</code> &rarr; <code>A.equals(C)</code></li>
<li>일관성(consistency)<code>A.equals(B) == A.equals(B)</code></li>
<li>null-아님<code>A.equals(null) == false</code></li>
</ol>

<br>
<br>
<br>

<h2>반사성(reflexivity)</h2><p>자기 자신을 equals의 매개변수로 호출 했을 때 true를 반환해야 한다.</p><pre><code>if(o == this)
	return true;
</code></pre>
<p>위 코드를 <code>equals</code>에 넣으면 쉽게 해결된다.</p>

<br>
<br>
<br>

<h2>대칭성(symmetry)</h2><p>대칭성이 위배된 잘못된 코드를 먼저 살펴보자</p><pre><code>//     대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
</code></pre>
<p><code>if (o instanceof String)</code> 이 부분이 바로 대칭성을 위배하는 부분이다.</p>
<p><code>CaseInsensitiveString.equals(String)</code>이 true이더라도 <code>String.equals(CaseInsensitiveString)</code>는 <code>String</code>은 <code>CaseInsensitiveString</code>를 알고 있지 않으므로 false를 반환하기 때문이다.</p>
<p>대칭성을 얻기 위해서는 다음과 같이 하면된다.</p><pre><code> // 수정한 equals 메서드 (56쪽)
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
</code></pre>
<p>간단히 다른 타입(여기서는 <code>String</code>)까지 지원하려는 것을 포기하면 된다.</p>

<p>또 다른 사례를 보자.
<code>ColorPoint</code>는 <code>Point</code>의 서브 클래스이며 <code>equals</code>를 다음과 같이 구현했다.</p><pre><code>@Override public boolean equals(Object o) {
    if (this == o) {
        return true;
    }

    if (!(o instanceof Point)) {
        return false;
    }

    Point p = (Point) o;
    return p.x == x && p.y == y;
}
</code></pre>

<p>그리고 아래는 <code>ColorPoint</code>의 <code>equals</code>이다.</p><pre><code>// 코드 10-2 잘못된 코드 - 대칭성 위배! (57쪽)
@Override public boolean equals(Object o) {
if (!(o instanceof ColorPoint))
return false;
return super.equals(o) && ((ColorPoint) o).color == color;
}
</code></pre>
<p><code>Point.eqauls(ColorPoint)</code>가 true를 반환할 수도 있지만 <code>ColorPoint.eqauls(Point)</code>는 <code>instanceof</code> 구문에 의해서(Point는 ColorPoint 타입이 아니므로) 반드시 false를 반환하며 대칭성을 위반하게 된다.</p>

<br>
<br>
<br>

<h2>추이성(transitivity)</h2><p>그럼 위 코드에서 type까지 고려한다면 어떨까?</p><pre><code>    // 코드 10-3 잘못된 코드 - 추이성 위배! (57쪽)
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        // o가 일반 Point면 색상을 무시하고 비교한다.
        if (!(o instanceof ColorPoint))
            return o.equals(this);

        // o가 ColorPoint면 색상까지 비교한다.
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
</code></pre>
<p>이렇게 구현한다면 <code>equals</code>의 <b>대칭성은 지켜진다</b>. 하지만 아래 코드를 보자.</p><pre><code>// 두 번째 equals 메서드(코드 10-3)는 추이성을 위배한다. (57쪽)
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
</code></pre>
<p>이 코드를 통해서 알 수 있는 것은 <b>추이성이 위반</b>된다는 것이다. <code>p1.equals(p2)</code>, <code>p2.equals(p3)</code>는 true 이지만 <code>p1.equals(p3)</code>는 false를 반환하게 된다.</p>
<p>또 하나 큰 문제점은 <code>Point</code>의 다른 서브 클래스의  <code>equals</code>를 위와 같이 구현할 경우 <code>StackOverflow</code>가 일어나게 된다.</p>
<p>그래서 추이성을 지키고자 <code>Point.equals</code>를 다음과 같이  변경할 수도 있다.</p><pre><code> // 잘못된 코드 - 리스코프 치환 원칙 위배! (59쪽)
@Override public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
</code></pre>
<p>이렇게 바꾼다면 앞서 말한 예시에서 추이성은 지켜진다. 하지만 객체 지향 프로그래밍에서 중요한 <b>리스코프 치환 원칙을 위반</b>하게 된다.</p><blockquote>
<p>리스코프 치환 원칙
어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.</p>
</blockquote>
<p>위 코드에서 <code>Point</code>의 서브 클래스들을 <code>Point</code>로 형변환 하여 <code>equals</code>를 호출한다면 좌표가 같더라도 <code>false</code>를 반환하게 될 것이다.</p>

<p>그렇다면, 어떻게 해야 <code>equals</code>의 일반 규약을 지킬 수 있을까?
일반적으로 instanceof 로 상위 클래스(인터페이스) 혹은 자신과 같은 클래스인지 비교하고 필드를 비교하면 되며 서브 클래스는 equals 메서드를 그대로 쓰면 된다. 하지만, <code>ColorPoint</code>와 같이 <b>값을 추가한 서브 클래스는 일반 규약을 지키면서 리스코프 치환 원칙까지 지키는 것은 불가능</b>하다.</p>
<p>이와 같은 경우 <b>상속 대신 컴포지션을 사용할 것을 책에서는 권장</b>한다.</p>
<h3>컴포지션</h3><p>다음과 같이 객체를 직접 상속하는 것이 아니라 내부에 필드로 가지고 있는 것을 말한다.</p><pre><code>// 코드 10-5 equals 규약을 지키면서 값 추가하기 (60쪽)
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
</code></pre>
<p>위와 같이 상속 대신 컴포지션 구현시 <code>Point</code> 뷰를 <code>asPoint</code>로 반환할 수 있으며 이를 통해 <code>equals</code>일반 규약을 지키는 것이 가능해진다.</p>



<h2>일관성(consistency)</h2><p>한 번 <code>A.equals(B)</code>가 true이면 그 다음에 <code>A.equals(B)</code>를 호출해도 true를 반환해야 한다는 것이다. 불변 객체라면 일관성이 항상 보장된다. 하지만 가변 객체라면 일관성이 깨질 수 있다.</p>
<p><code>URL</code>의 경우 호스트의 IP 주소를 이용하여 비교하기 때문에 Virtual Host의 경우 일관성이 보장되지 않는다. 네트워크를 통해서 IP 주소를 가지고 오기 때문이다.</p>
<p>따라서, <code>URL</code>과 같이 <code>신뢰할 수 없는 자원</code>을 <code>equals</code>의 판단에 끼어들게 해서는 안된다. 메모리에 있는 객체를 이용한 결정적인(deterministic) 계산만 수행해야 한다.</p>

<br>
<br>
<br>

<h2>null-아님</h2><p>null이 인자로 넘어오면 false를 반환해야 한다는 것이다. <code>instanceof</code>구문을 사용하면 null 일 경우 false가 반환되므로 <code>instanceof</code>를 활용하자.</p>

<br>
<br>
<br>

<h2>일반 규약을 지키는 구현방법</h2>
<ol>
<li>자기 자신인지 확인한다.<pre><code>if (this == o) {
 return true;
}
</code></pre></li>
<li>타입 비교<pre><code>if (!(o instanceof Point)) {
 return false;}
</code></pre></li>
<li>형 변환<pre><code>Point p = (Point) o;
</code></pre></li>
<li>핵심 필드들이 모두 일치하는지 하나씩 검사한다.<pre><code>return p.x == x && p.y == y;
</code></pre></li>
</ol><p>여기서 유의할 점은 다음과 같다.</p><ul>
<li>float과 double 은 == 비교가 아니라, 각각 정적 메서드인 <code>Float.compare(float, float)</code>과 <code>Double.compare(double, double)</code>로 비교한다.</li>
<li>Reference type은 그 type이 가지고 있는 equals를 호출한다.</li>
<li>Synchronized Lock(동기화 용 락) 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다.</li>
<li>null일 수도 있는 필드를 비교하려면 <code>Object.equals(Object, Object)</code>로 비교한다.</li>
</ul>
<p>위를 기반으로 다음과 같이 <code>equals</code>를 override할 수 있다.</p><pre><code>@Override 
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }

    if (!(o instanceof Point)) {
        return false;
    }

    Point p = (Point) o;
    return p.x == x && p.y == y;
}
</code></pre>
<br>
<br>
<br>
<h2>실제</h2><p>구현하기가 너무 복잡하다. 따라서 우리는 주로 Tool을 사용한다.</p><ol>
<li>AutoValue쓰는 방법이 조금 까다롭다. 컴파일시에 생성되는 코드를 기반으로 하기 때문에, 없는 클래스를 참조해야한다. &rarr; 즉, 너무 invasive(침투적)이다.</li>
<li>Lombok상대적으로 비침투적이다.자바 11버전일 경우 추천</li>
<li>record자바 14버전에 처음 들어왔던 기능자바 17버전 부터 추천(record에 적절하지 않은 경우면 Lombok)</li>
<li>인텔리제이에서 만들기약간 지저분하다. 필드가 늘어나면 매번 다시 만들어야 한다.</li>
</ol>
<br>
<br>
<br>
<h2>주의사항</h2><ul>
<li><code>equals</code>를 재정의할 때 <code>hashCode</code>도 반드시 재정의하자.</li>
<li>너무 복잡하게 구현하지 말자.</li>
<li>Object가 아닌 타입의 매개변수를 받는 equals 메서드는 무용지물이다!</li>
</ul>