<h1>8. finalizer와 cleaner 사용을 피하라</h1><br><br>
<h2>개요</h2><p>자원 회수를 위해서 사용할 수 있는 <code>finalizer</code>와<code>cleaner</code>는 <b>즉시 수행된다는 보장이 없으며 상황에 따라 위험할 수도 있어</b> 불필요하다.</p><br>
<br>
<br>
<br>
<br>
<h2>Finalizer</h2><br>
<p>다음과 같이 클래스에 <code>Object.finalize</code> 메서드를 오버라이딩 해주면 된다.</p><pre><code>@Override
protected void finalize() throws Throwable {
    System.out.print("");
}
</code></pre><p>자바 9부터 Deprecated 되었다. 대안으로 <code>AutoCloseable</code>, <code>Cleaner</code>, <code>WeakReference</code>, <code>PhantomReference</code> 등을 제시하고 있다. (<code>AutoCloseable</code>가 제일 낫다고 한다)</p><br>
<p>또한 상속을 악용한 <code>Finalizer</code> 공격이 일어날 수도 있다.</p><br>
<br>
<br>
<br>
<br>
<h2>Cleaner</h2><br>
<p>다음과 같이 static class로 구현하면 된다.</p><pre><code>// 해당 객체(this)를 내부에서 절대로 참조해선 안된다.
public static class ResourceCleaner implements Runnable {

    private List&lt;Object&gt; resourceToClean;

    public ResourceCleaner(List&lt;Object&gt; resourceToClean) {
        this.resourceToClean = resourceToClean;
    }

    @Override
    public void run() {
        resourceToClean = null;
        System.out.println("cleaned up.");
    }
}
</code></pre><br>
<p>이렇게 구현한 클래스를 다음과 같이 사용하면 된다.</p><pre><code>Cleaner cleaner = Cleaner.create();

// Cleaner의 대상이 될
List&lt;Object&gt; resourceToCleanUp = new ArrayList&lt;&gt;();
BigObject bigObject = new BigObject(resourceToCleanUp);

// Cleaner에 등록, 위에서 Runnable을 구현한 static 클래스를 2번째 매개변수로 넣어준다.
cleaner.register(bigObject, new BigObject.ResourceCleaner(resourceToCleanUp));

//bigObject의 참조가 해제되면 GC가 일어날 때 등록된 자원의 메모리가 해제된다.
</code></pre><br>
<br>
<br>
<br>
<br>
<h2>권장 방법</h2><p><code>AutoCloseable</code>과 <code>try-with-resources</code>를 사용하는 것이다. <code>Cleaner</code>는 안전망으로서 활용할 수도 있다. (클라이언트 코드에서 실수로 <code>try-with-resources</code> 를 사용하지 않을 경우 GC가 수행될 기회를 줄 수 있기 때문)</p><br>
<br>
<h2>AutoCloseable</h2><p><code>AutoCloseable</code> 인터페이스를 활용하기 위해서 <code>AutoCloseable.close</code> 메서드를 재정의해야 한다.<br>
<code>close</code>메서드는 기본 스펙에 다음과 같이 throws Exception이 정의되어 있다.</p><pre><code>void close() throws Exception
</code></pre><br>
<p>하지만 오버라이딩 할 때 반드시 throws Exception을 해야하는 것이 아니다.</p><ul>
<li>만약, <b>예외를 던져야 하는 상황이라면 throws Exception 대신에 throws {구체적인 예외 클래스}로 구현할 것을 권장한다.</b> &rarr; 클라이언트 코드에 예외 처리 책임 전가</li>
<li>예외를 내부에서 처리할 수도 있다(try-catch 구문으로).
<ul>
<li>예외처리시 별다른 할 일이 필요하지 않을 때는 catch 구문에 단순히 throw new RuntimeException(e)를 추가해주는 것도 좋다.</li>
</ul>
</li>
</ul><pre><code>try {
    reader.close();
} catch (IOException e) {
    throw new RuntimeException(e);
}
</code></pre><br>
<ul>
<li>가급적이면 <code>close</code>메서드는 멱등성을 가지도록 권장한다. (클라이언트가 같은 인스턴스의 <code>close</code>를 여러번 호출할 가능성이 있기 때문)</li>
</ul><br>
<br>
<br>
<h2>Finalize 공격에 대비</h2><ol>
<li>클래스를 final로 선언한다.</li>
<li><code>finalize</code>를 미리 final로 선언해서 오버라이딩을 막는다.</li>
<br>
<pre><code>   @Override
final protected void finalize() throws Throwable {
    super.finalize();
}
</code></pre><br>
<li>호출을 방어하고자 하는 메서드에 검증 절차를 추가한다.</li>
<br>
<pre><code>public void transfer(BigDecimal amount, String to) {
    if(this.accountId.equals("푸틴"))
        throw new IllegalArgumentException("푸틴은 계정을 막습니다.");

    System.out.printf("transfer %f from %s to %s\n", amount, accountId, to);
}
</code></pre>
</ol>
<br>
<p>참고:  <a href='https://self-learning-java-tutorial.blogspot.com/2020/03/finalizer-attack-in-java.html'>Finalizer Attack in Java</a></p><br>
<br>
