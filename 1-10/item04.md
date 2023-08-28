<h1>5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.</h1>
<h2>의존 객체 주입(DI)</h2><p><b>사용하는 자원에 따라 동작이 달라지는 클래스</b>에 사용하기 적합하다.</p><br>
<br>
<h3>문제 상황</h3><ol>
<li>자원을 직접 명시하게 되면 Mocking을 할 수가 없게된다. (엄밀히 말하면 가능은 하지만 번거롭고 바람직하지 않다.)</li>
</ol><br>
<pre><code>@Test
void isValid() {
	assertTrue(SpellChecker.isValid("test"));
}

//SpellChecker.class

// 자원을 직접 명시하는 코드
private static final Dictionary dictionary = new DefaultDictionary();
</code></pre><p>이미 자원이 클래스가 로드되는 시점에서 무조건 생성되며 변경할 수도 없다.</p><br>
<ol>
<li>유연성과 재사용성이 떨어진다.</li>
</ol><br>
<p>예제에 나온 <code>SpellChecker</code>가 <code>DefaultDictionary</code>가 아닌 <code>KoreanDictionary</code>나 <code>EnglishDictionary</code>를 사용하고 싶다면 어떻게 해야 할까? 기존처럼 리소스를 고정해놓는 방식을 고수한다면 <code>KoreanSpellChecker</code>, <code>EnglishSpellChecker</code> 처럼 코드가 중복된 클래스를 별도로 만들어야 한다. 게다가, 런타임 도중에 <code>Dictionary</code>를 바꾸기도 어려울 것이다.</p><br>
<br>
<h3>DI</h3><p>자원을 <b>인터페이스로 갖고 있으며 객체를 외부에서 주입하는 경우</b> 코드가 유연하고 재사용성이 높아지게 된다. 게다가 Mocking이 가능하여 테스트도 용이해진다.</p><br>
<h3>변형</h3><p>책에서 생성자에 <code>자원 팩터리</code>를 넘겨 주는 것이 의존 객체 주입의 쓸만한 변형이라고 한다.</p><br>
<pre><code>public SpellChecker(Supplier&lt;Dictionary&gt; dictionarySupplier) {
    this.dictionary = dictionarySupplier.get();
}
</code></pre><p>그리고 위와 같이 Supplier<T> 인터페이스를 활용하는 것이 완벽한 예라고 한다. (하지만, 자원팩토리 인터페스를 직접 구현해서 넣는 것도 가능하다.)</p><br>
<br>
<h3>유의할 점</h3><p>의존 객체 주입은 직접 구현하면 코드를 어지럽게 만들 수 있으므로, 프레임워크를 활용하여 코드를 간결하게 유지하자.</p><br>
<br>
<p>결국 DI는 유연함, 재사용성, 테스트 용이성 에서 강점을 얻는다.<br>
+) 유연함이란?</p><br>
<p>개인적인 정의: 여러가지 상황과 변화에 대처할 수 있는 능력을 의미한다.<br>
예: SpellChecker 에서 사용되는 Dictonary 의 기능을 바꾸고 싶다면? SpellChecker와 동일한 클래스를 만들 것이 아니라 SpellChecker에 다른 Dictionary 를 만들어서 주입함으로써 변화에 쉽게 대응할 수 있다.<br>
즉 OPC(수정에는 닫혀 있고 확장에는 열려있다.)</p><br>
