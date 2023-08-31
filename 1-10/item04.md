<h1>4. 인스턴스화를 막으려거든 private 생성자를 사용하라</h1><br>
<ul>
<li>간혹 인스턴스화를 막는 것이 바람직한 유틸리티성 클래스들이 있다.
<ul>
<li><code>public static</code> 메서드를 가진 클래스들</li>
<li>로직을 도와주는 성격의 클래스</li>
<li>ex) <code>org.springframework.util.StringUtils</code>, <code>java.lang.Math</code>, <code>java.util.Arrays</code>, <code>java.util.Collections</code>와 같은 유틸리티 클래스가 있다.</li>
</ul>
</li>
<li>유틸리티 클래스는 <code>public static</code> 메서드만 제공하기 때문에 인스턴스화가 필요하지 않으며, 인스턴스를 통한 메서드 호출은 오히려 인스턴스 메서드와 정적 메서드간의 혼동을 불러 일으킨다. 따라서, 인스턴스화를 방지하는 것이 바람직하다.<br><br></li>
<li>인스턴스화를 방지하기 위해서 <code>abstract</code>로 클래스를 만들 수 있다.</li>
<li>하지만 이 방법은 인스턴스화를 방지할 수 없다.
<ul>
<li>자바 컴파일러는 생성자를 명시하지 않으면 기본 <code>public</code> 생성자를 만든다.</li>
<li>만약, 하위클래스를 만들어 상속하게 되면 해당 클래스의 인스턴스를 만들 수 있게된다.</li>
</ul>
<br><br></li>
<li>그래서  대안은 private 생성자를 명시적으로 선언하는 것이다.</li>
</ul><br>
<pre><code>	private UtilityClass() {}
</code></pre><br>
<ul>
<li>하지만, 내부에서 private 생성자를 호출할 수도 있기 때문에 내부에서의 인스턴스 생성까지 막기 위해서는 기본 생성자에서 에러(<code>AssertionError</code>)  <code>throw</code>하도록 하면된다.</li>
</ul><br>
<pre><code>    private UtilityClass() {
        throw new AssertionError();
    }
</code></pre><ul>
<li>하지만, 생성자가 분명 존재하는데 호출을 막는 코드는 직관적이지 않다. 따라서, 문서화를 해서 왜 private 생성자를 만들었는지 명시하는 것이 바람직하다.</li>
<li>게다가 이 방식은 <b>상속을 불가능</b>하게 하는 효과도 있다. (자식 클래스에서 부모 클래스의 생성자를 호출할 수 없기 때문에)</li>
<li>사실, Spring 의 많은 유틸리티 클래스들은 이 방법을 쓰지 않고 있다.</li>
</ul><br>
<h2>결론</h2><ul>
<li>유틸리티 클래스는 private 생성자를 명시함으로써 인스턴스화를 방지할 수 있다.</li>
<li>인스턴스화를 완전히 차단하기 위해서 <code>Error</code>를 <code>throw</code>하도록 할 수 있다.</li>
<li>private 생성자를 사용한다면 문서화를 통해 목적을 명시하자.</li>
</ul>