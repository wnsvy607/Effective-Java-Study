<h1>18. 상속보다는 컴포지션을 사용하라.</h1>
<br>
<br>
<br>
<h2>개요</h2><p>인터페이스 상속이 아닌 구현 상속보다는 컴포지션을 사용하는 것이 낫다.</p>
<br>
<br>
<h2>상속의 문제점</h2><blockquote>
<p>메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.</p>
</blockquote>
<p>패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 일은 위험하다. 상위 클래스에서 제공하는 메서드의 구현이 바뀌거나 새로운 메서드가 생긴다면 하위 클래스가 의도한 대로 동작하지 않을 확률이 높다. 결국 상속하려면 상위 클래스의 내부 구현을 알아야 하며 내부 구현이 바뀌면 변경에 대응해야 한다. 하지만 변경을 인지하기도 어렵다.</p>
<p>다음 코드는 상속의 잘못된 예시를 보여주는 클래스다.</p><pre><code>public class InstrumentedHashSet&lt;E&gt; extends HashSet&lt;E&gt; {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection&lt;? extends E&gt; c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
</code></pre><p>Set에 원소가 추가할 때마다 카운트를 하나씩 하는 원소다. (책에서는 성능을 최적화를 위해서 이렇게 추가된 원소를 카운트하는 식으로 구현할 수도 있다고 한다.) <code>HashSet</code>을 상속 받아 <code>add</code>와 <code>addAll</code> 메서드를 재정의해서 값이 추가될 때마다 카운트를 더 하도록 했다.</p>
<br>
<br>

<p>그럼 아래의 메서드는 어떻게 동작할까?</p>
<pre><code>    public static void main(String[] args) {
        InstrumentedHashSet&lt;String&gt; s = new InstrumentedHashSet&lt;&gt;();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
</code></pre><p>3이라고 생각하겠지만 사실은 6이 출력된다. 왜냐하면 <code>HashSet.addAll</code>은 내부적으로 <code>add</code>를 다시 호출하는데, <code>add</code> 역시 <code>InstrumentedHashSet</code>에 의해서 재정의 되었기 때문이다. 이처럼 자신의 다른 부분을 사용하는 ‘자기사용(self-use)’ 여부는 철저히 클래스의 내부 구현 방식이다. 게다가 내부 구현 방식에 맞춰 올바르게 동작하도록 <code>addAll</code>메서드를 수정해서 오버라이딩해도 <code>HashSet</code>의 구현이 바뀐다면 다시 변경해야만 한다. 즉, 재정의를 하기 위해서는 상위 클래스의 내부 구현을 상세히 알아야만 하는 것이다. <b>다시 말하자면 캡슐화가 깨지는 것이다.</b>
게다가 상위 클래스에서 새로운 메서드가 정의될 때, 하위 클래스에서 만든 메서드와 시그니처가 같다면 컴파일 오류가 발생할 수도 있다.</p>
<br>
<br>
<br>
<h2>컴포지션</h2><p>상속으로 인한 문제점을 최소화하기 위해서는 컴포지션을 사용하자. 우선, 새로운 클래스를 만들고 상속하고자 하는 클래스를 <code>private</code> 필드로 선언한다. 그리고 새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드들을 호출해 그 결과를 반환하도록 만들자. 이것을 전달(forwarding)방식이라고 하며 이런 메서드들을 전달 메서드(forwarding method)라고 한다. 컴포지션의 <b>가장 큰 특징은 기존 클래스의 구현이 바뀌거나, 새로운 메서드가 생기더라도 아무런 영향을 받지 않는다는 것이다.</b>
책에서는 아래와 같이 <b>래퍼 클래스와 전달 클래스</b>를 만들어서 사용하는 예시가 나왔다.</p>
<p>래퍼 클래스</p><pre><code>// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
public class InstrumentedSet&lt;E&gt; extends ForwardingSet&lt;E&gt; {
    private int addCount = 0;

    public InstrumentedSet(Set&lt;E&gt; s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection&lt;? extends E&gt; c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}
</code></pre>
<p>전달 클래스(컴포지션, 위임, 데코레이터 패턴)</p><pre><code>// 코드 18-3 재사용할 수 있는 전달 클래스 (118쪽)
public class ForwardingSet&lt;E&gt; implements Set&lt;E&gt; {
    private final Set&lt;E&gt; s;
    public ForwardingSet(Set&lt;E&gt; s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator&lt;E&gt; iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection&lt;?&gt; c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection&lt;? extends E&gt; c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection&lt;?&gt; c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection&lt;?&gt; c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public &lt;T&gt; T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
</code></pre>
<p>그럼 아래 코드는 어떻게 동작할까?</p><pre><code>public static void main(String[] args) {
    InstrumentedSet&lt;String&gt; s = new InstrumentedSet&lt;&gt;(new HashSet&lt;&gt;());
    s.addAll(List.*of*("틱", "탁탁", "펑"));
    System.*out*.println(s.getAddCount());
}
</code></pre><p>3이 나오게 된다 왜냐하면 내부 구현과 상관없이 <code>addAll</code>에서 호출되는 <code>Set.add</code>는 재정의 되지 않았기 때문이다. 이렇게 구현한다면 <b>기존 클래스 내부 구현방식에 전혀 영향을 받지 않게되며 캡슐화를 지킬 수 있게 된다.</b> 게다가 위 전달 클래스에서는 인터페이스를 구현했기 때문에 구현체(<code>HashSet</code>)에 새로운 기능이 생겨도 무관하며 인터페이스에 새로운 API가 추가되면 곧 바로 인지가 가능하다.</p>
<br>
<br>
<br>
<h2>결론</h2><p>상속보다는 컴포지션을 사용하자.</p>