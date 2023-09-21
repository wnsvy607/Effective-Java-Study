<h1>13. Clone 재정의는 주의해서 진행하라.</h1>
<br>
<br>
<br>
<h2>구현 - 불변 객체</h2><ol>
<li><code>Cloenable</code>은 텅 비어 있는 믹스인 인터페이스지만 <code>clone</code>을 이용하려면 반드시 구현(implements)해주어야 한다.</li>
<li>그 다음 <code>clone</code> 메서드를 <code>super.clone</code>을 사용해 오버라이딩해줘야 한다.</li>
<li>접근 제한자는 public 으로 설정, 반환 타입은 자신의 클래스로 변경한다.</li>
</ol>
<br><br>
<h2>Clone 규약</h2><ol>
<li><code>x.clone() != x</code><code>clone</code> 반드시 원본과 다른 인스턴스를 반환해야 한다. (레퍼런스 자체가 달라야한다.)</li>
<li><code>x.clone().getClass() == x.getClass()</code>가 반드시 true이여야 한다.원본의 클래스와 동일한 인스턴스를 반환해야 한다.</li>
<li><code>x.clone().equals(x)</code>가 true가 아닐 수도 있다.복사를 한 인스턴스가 원본과 달라야만 한다면 true가 아닐수도 있다.</li>
</ol>
<br><br>
<h2>구현 - 가변 객체</h2><ol>
<li>불변 객체의 구현 과정을 동일하게 수행한다.</li>
<li>배열인 필드의 경우 <code>clone</code>을 따로 호출해서 필드에 할당해주어야 한다. 문제는 이렇게 해도 배열의 원소들이 레퍼런스 타입이라면 <b>배열자체는 복사</b>가 되지만 <b>배열의 원소는 Shallow Copy되어 존재</b>하게 된다. 따라서 <b>이런 경우 배열 내부 원소들까지 Deep Copy를 해줘야</b> 한다.</li>
</ol>
<br><br>
<h2>유의할 점</h2><ul>
<li>자바는 <code>Overriding</code>할 때, 리턴 타입을 <code>Overriding</code>대상이 되는 메서드의 리턴 타입의 하위 타입으로 지정해도 <code>Overriding</code>으로 인정된다. &larr; 타입 캐스팅을 하지 않아도 되는 장점이 있다.이것을 공변환 이라고 하는데 <code>clone()</code>을 <code>Overriding</code>할 때 사용하면 좋다고 한다.</li>
<li><code>Object.clone</code>은 <code>CloneNotSupportedException</code>라는 Checked Exception을 throw 하는데, 바람직한 설계라고 보기는 어렵다. 왜냐하면 해당 예외가 발생한다고 해서 할 수 있는 일이 없기 때문이다.참고로 해당 예외는 <code>clone</code>이 호출된 클래스가 <code>Cloneable</code>을 구현(implement) 하지 않으면 발생한다.</li>
<li><code>clone</code>을 구현할 때 <code>super.clone()</code>을 호출하지 않고 단순히 생성자를 이용할 수도 있다고 생각할 수도 있지만 반드시 <code>super.clone()</code>을 호출해주어야 한다.왜냐하면 상속에서 문제가 발생하기 때문이다. 하위 타입에서 <code>clone</code>을 오버라이딩 할때 다음과 같이 구현할 수 있다.<pre><code>// Item.java
public class Item implements Cloneable {
  private String name;

  @Override
  public Item clone() {
      Item item = new Item();
      item.name = this.name;
      return item;
  }
}
// SubItem.java
public class SubItem extends Item implements Cloneable {

  private String name;

  @Override
  public SubItem clone() {
      return (SubItem)super.clone();
  }
</code></pre></li>
</ul><p>위와 같이 두 클래스가 있을 때 <code>clone</code>을 호출하면 어떻게 될까? <code>ClassCastException</code>이 발생한다. 이유는 자바에서는 상위타입을 하위타입으로 <code>Cast</code>할 수가 없기 때문이다.(반대는 가능하다.)
좀 더 구체적으로, <code>Item</code>의 <code>super.clone()</code> 메서드는 <code>Item</code>을 반환하는데, 이것을 <code>SubItem</code>으로 캐스팅할 수 없기 때문에 <code>ClassCastException</code>이 발생하는 것이다. 따라서 항상 <code>super.clone()</code>을 호출해야 한다.</p>
<ul>
<li>객체 생성에 관여하는 메서드들(이를테면 생성자, <code>clone</code>이 있다)은 하위 클래스에서 오버라이딩하지 못하도록 <code>final</code> <code>private</code>으로 막거나 엄격한 룰을 적용하도록 강제해야 한다. 왜냐하면 하위 클래스에서 오버라이딩 하면 동작이 바뀔 가능성이 있기 때문이다.</li>
<li>일반적으로 상속용 클래스에 <code>Cloneable</code> 인터페이스를 사용하지 않는 것이 좋다. 이렇게 하면 <code>clone</code>을 해당 클래스의 상속을 사용하려는 프로그래머에게 수 많은 고민을 안겨주기 때문이다. 그래서 방법이 2가지 있다.
<ol>
<li>미리 <code>clone</code>을 메서드로 정의한다.</li>
<li>아예 <code>protected final</code>이며 호출 시 Exception을 throw하는  <code>clone</code> 메서드를 만들어 오버라이딩을 못하게 만들 수 있다.</li>
</ol>
</li>
<li><code>clone</code>을 구현할 때 객체 자체는 <code>super.clone</code>으로 복사를 하고 그 객체의 모든 필드는 그 필드의 클래스가 제공하는 외부로 노출된 (public) 고수준 API (이를테면 <code>Map.put</code>)를 통해서 데이터를 새로 만드는 방법이 있다. 안정적이긴 하나 저수준에 비해 다소 느릴 가능성도 있다. &larr; 대부분의 경우에 문제가 되지 않는다고 한다.</li>
<li>만약에 Thread-safety를 보장해야 하는 클래스라면 <code>synchronized</code>를 써야 한다.</li>
</ul>
<br><br>
<h2>실제</h2><p>너무 사용하기 복잡해서 복사 전용 생성자나 정적 팩토리 메서드를 사용하는 것이 바람직하다.</p><ol>
<li>생성자는 우리가 어떻게 동작하는 지 알지만 <code>clone</code>은 내부 동작이 너무 불분명하다.</li>
<li>private final이면서 레퍼런스 타입인 필드의 경우 <code>clone</code> 으로  Deep copy가 불가능하다.</li>
</ol>
<p>생성자의 정적 팩토리 메서드의 장점은 다음과 같다.</p><ol>
<li>상위 타입(인터페이스 등)을 인자로 받아 변환하여 복사하는 것이 가능하며 이것을 변환 생성자(conversion constructor), 변환 팩터리(conversion factory)라고 한다.</li>
<li>private final 필드까지 Deep copy가 가능하다.</li>
</ol>
<br><br>
<h2>결론</h2><ul>
<li><code>clone</code>은 사용법이 너무 복잡하다.</li>
<li>복사 전용 생성자나 정적 팩토리 메서드를 사용하자</li>
</ul>