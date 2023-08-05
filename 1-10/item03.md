<h1>아이템 3.private 생성자나 열거 타입으로 싱글톤임을 보증하라.</h1><h2>싱글톤을 사용하는 경우</h2><blockquote>
<p>크게 2가지 경우에 싱글톤을 사용하면 된다.</p>
</blockquote><br>
<ol>
<li>싱글톤을 사용하면 좋은 경우
<ul>
<li>DB 커넥션 등 인스턴스 생성 비용이 비싼 경우</li>
<li>객체를 재사용하는 것이 용이할 때</li>
</ul>
</li>
<li>반드시 사용해야 하는 경우
<ul>
<li>환경설정 등 애플리케이션 전반에 하나만 존재해야 하는 경우</li>
</ul>
</li>
</ol><br>
<br>
<h2>Private 생성자</h2><p>이펙티브 자바에서는 private 생성자를 기본으로 싱글톤을 구현하는 방법을 소개하고 있다.<br>
그 중 첫 번째 방법인 public static final 멤버 변수를 사용하는 방법은 다음과 같다.</p><pre><code>public static final Elvis INSTANCE = new Elvis();
</code></pre><p>즉, 필요 할 때마다 <code>Elvis. INSTANCE</code>를 직접 호출하면 된다.</p><br>
<br>
<h3>문제점</h3><ol>
<li>테스트하기 어려운 구조</li>
</ol><br>
<p>싱글톤 클래스를 사용하는 클라이언트 코드의 테스트가 어려워진다. 인터페이스라면 Mock 객체를 만들 수 있지만, 인터페이스 기반이 아니라면 Mocking이 어렵기 때문이다.</p><br>
<ol>
<li>리플렉션을 통해서 private 생성자를 호출할 수 있다. 즉, 리플렉션으로 새로운 인스턴스가 생성되어 싱글톤이 훼손(공격)될 수 있다.</li>
</ol><br>
<p>&rarr; 생성자에 인스턴스 생성후에 생성자가 호출되면 예외를 던지는 코드를 추가할 수 있다.</p><pre><code>public static final Elvis INSTANCE = new Elvis();
private static boolean created;

private Elvis() {
    if (created) {
        throw new UnsupportedOperationException("can't be created by constructor.");
    }

    created = true;
}
</code></pre><p>(하지만 간결함이라는 장점을 잃는다.)</p><br>
<ol>
<li>역직렬화 시 새로운 인스턴스가 생성된다. (의도와 다르게 동작한다)</li>
</ol><br>
<blockquote>
<p>직렬화(Serialize)란?</p>
</blockquote><br>
<p>직렬화란 객체 정보를 어딘가에 저장하는 것, 역직렬화는 어딘가에 저장되어 있는 직렬화된 객체 정보를 읽어 오는 것이다.</p><br>
<p>Spring을 사용하거나 readResolve() 메서드를 구현하여 해결 가능하다.<br>
readResolve()는 Overriding이 아니지만 Overriding처럼 사용하면 된다.</p><pre><code>private Object readResolve() {
    return INSTANCE;
}
</code></pre><br>
<br>
<h2>정적 팩터리 메서드</h2><p>private 생성자를 사용하는 두 번째 방법으로 위의 public static final 멤버 변수를 사용하는 방법과 같은 단점을 가지고 있다.</p><pre><code>private static final Elvis INSTANCE = new Elvis();
private Elvis() { }
public static Elvis getInstance() { return INSTANCE; }
</code></pre><br>
<br>
<br>
<h3>장점</h3><ol>
<li>정적 메서드를 통해 인스턴스를 접근하게 하면, 싱글톤을 사용할지 매번 다른 인스턴스를 생성할 지 클라이언트 코드의 변경없이 변경할 수 있다.<br></li>
<li>원한다면 제네릭 싱글톤 팩터리를 사용할 수 있다.</li>
</ol><br>
<pre><code>public class MetaElvis&lt;T&gt; {

    private static final MetaElvis&lt;Object&gt; INSTANCE = new MetaElvis&lt;&gt;();

    private MetaElvis() { }

    @SuppressWarnings("unchecked")
    public static &lt;E&gt; MetaElvis&lt;E&gt; getInstance() { return (MetaElvis&lt;E&gt;) INSTANCE; }

    public void say(T t) {
        System.out.println(t);
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
</code></pre><p>이렇게 하면 같은 싱글톤 인스턴스를 사용하지만 제네릭 타입에 따라 용도를 다르게 사용할 수 있다. <code>equals()</code>비교시 true 를 반환한다.</p><br>
<pre><code>public static void main(String[] args) {
    MetaElvis&lt;String&gt; elvis1 = MetaElvis.getInstance();
    MetaElvis&lt;Integer&gt; elvis2 = MetaElvis.getInstance();
    System.out.println(elvis1);
    System.out.println(elvis2);
    elvis1.say("hello");
    elvis2.say(100);
}
</code></pre><br>
<pre><code>// 출력결과
me.whiteship.chapter01.item03.staticfactory.MetaElvis@7f690630
me.whiteship.chapter01.item03.staticfactory.MetaElvis@7f690630
hello
100
</code></pre><br>
<br>
<ol>
<li>정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다.</li>
</ol><br>
<p><code>Elvis::getInstance</code></p>