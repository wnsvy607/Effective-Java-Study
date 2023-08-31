<h1>7. 다 쓴 객체 참조를 해제하라.</h1>
<br>
<br>
<h1>메모리 누수 발생을 주의하자</h1><p>메모리 관리를 직접하는 경우 GC가 완벽하게 동작할 것을 기대하면 안된다 코드 내부에 더 이상 쓰이지 않지만 참조해제가 안된 객체들이 계속해서 존재할 가능성이 있다.</p><br>
<br>
<h2>주요 원인</h2><ol>
<li>Stack</li>
<li>Cache</li>
<li>Listener</li>
</ol><br>
<p>공통점: 모두 내부에 객체를 쌓아두는 공간이 있다!</p><br>
<blockquote>
<p>일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.
이펙티브 자바 38p</p>
</blockquote><br>
<br>
<br>
<h2>대안</h2><br>
<h2>사용하지 않는 변수에 대해 명시적으로 null을 삽입해준다.</h2><pre><code>public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
</code></pre><p>하지만, 이 방법도 바람직한 것은 아니라고 책에서는 다음과 같이 말한다.</p><blockquote>
<p>객체 참조를 null 처리하는 일은 예외적인 경우여야 한다. 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.</p>
</blockquote><br>
<br>
<br>
<br>
<br>
<br>
<h2><b>WeakHashMap</b>을 사용한다.</h2><p>일반 <code>HashMap</code>은 키가 외부에서 참조하지 않아도 <code>Map</code>내부에 존재 하므로 해당 Key가 GC의 대상이 되지 않지만, <code>WeakHashMap</code>은 키가 외부에서 <b>강하게 참조(Strong Reference)</b>하는 곳이 없으면 바로 GC의 대상이 된다.</p><br>
<p><b>주의할 점!</b> WeakReference<T>는 List<T> 등에 넣은 다음 참조를 null로 바꿔도 GC의 대상이 되지 않는다. 즉, WeakReference<T> 를 위한 List를 따로 커스텀해서 구현해야 한다.</p><br>
<p>레퍼런스 종류</p><ul>
<li>Strong, Soft, Weak, Phantom</li>
</ul><br>
<br>
<h3>Soft Reference</h3><p>Strong Reference가 없고 Soft Reference만 있으며 <b>메모리가 필요할 경우에만</b> GC의 대상이 된다.</p><br>
<br>
<h3>Phantom Reference</h3><p><code>ReferenceQueue&lt;T&gt;</code>로 사용 가능하다. PhantomReference만 남을 경우 객체는 사용할 수 없지만 ReferenceQueue에 들어와 있다.<br>
용도는?</p><ol>
<li>자원 정리</li>
<li>언제 객체의 메모리가 해제되는 지 알 수 있다.</li>
</ol><br>
<br>
<h3>유의점</h3><ul>
<li>맵의 엔트리를 맵의 Value가 아니라 Key에 의존해야 하는 경우에 사용할 수 있다.<br>(Key가 없어질 때 Value가 없어져도 상관 없는 경우)</li>
<li><code>Interger</code>같은 Primitive Type의 래퍼 클래스나 <code>String</code>타입의 경우 JVM 내부에 캐싱이 되므로 따라서, 이런 경우에는 레퍼런스하는 변수를 null로 바꿔도 지워지지 않는다.</li>
<li>하지만, 이 방식은 자원이 언제 반납되는 지 애매하기 때문에 명시적으로 처리하는 편이 낫다. 따라서 Soft, Weak, Phantom 등은 극히 드물게 쓰인다.</li>
</ul><br>
<br>
<br>
<br>
<h2><b>백그라운드 스레드</b>를 활용 한다.</h2><ul>
<li>백그라운드 스레드를 통해 애플리케이션이 동작하는 동안 더 이상 안쓰는 객체들을 정리할 수 있다.</li>
</ul><br>
<h3>ExecutorService</h3><ul>
<li>쓰레드를 만드는 작업은 생성비용이 매우 크다</li>
<li>그래서 <b>쓰레드 풀</b>을 만들어서 사용하는데, <code>ExecutorService</code>로 쓰레드 풀을 편리하게 만들 수 있다. 쓰레드의 개수를 매개 변수로 줄 수 있다.</li>
</ul><pre><code>	ExecutorService service = Executors.newFixedThreadPool(10);	
	Future&lt;String&gt; submit = service.submit(new Task());

	static class Task implements Callable&lt;String&gt; {

    @Override
    public String call() throws Exception {
        Thread.sleep(2000L);
        return Thread.currentThread() + " world";
    }
</code></pre><br>
<ul>
<li>CPU-intensive한 작업은 쓰레드가 많아도 큰 효율을 볼 수 없다.</li>
<li>반면 IO-intensive한 작업은 쓰레드가 많을 때 효율적일 수 있다. &rarr; 적절한 쓰레드의 갯수는 직접 튜닝해봐야 알 수 있다.</li>
<li><code>Executors.newCachedThreadPool()</code> 필요할 때마다 쓰레드를 생성하는 쓰레드 풀, 오래 사용되지않은 쓰레드는 제거된다. 쓰레드가 무한정 늘어날 수 있기 때문에 반드시 자원낭비를 유의해야 한다!</li>
<li><code>Executors.newSingleThreadExecutor()</code> 쓰레드 하나만을 사용하는 쓰레드 풀</li>
<li><code>Executors.newScheduledThreadPool()</code>는 내부적으로 특이한 자료구조를 사용해서 스케쥴링을 감안해서 순서가 바뀔 수 있다.</li>
</ul><br>
<h3>예시</h3><pre><code>Runnable removeOldCache = () -&gt; {
    System.out.println("running removeOldCache task");
    Map&lt;CacheKey, Post&gt; cache = postRepository.getCache();
    Set&lt;CacheKey&gt; cacheKeys = cache.keySet();
    Optional&lt;CacheKey&gt; key = cacheKeys.stream().min(Comparator.comparing(CacheKey::getCreated));
    key.ifPresent((k) -&gt; {
        System.out.println("removing " + k);
        cache.remove(k);
    });
};

executor.scheduleAtFixedRate(removeOldCache, 1, 3, TimeUnit.SECONDS);
</code></pre><p>위와 같은 코드로 백그라운드 스레드에서 더 이상 사용하지 않는 객체를 정리할 수 있다.</p><br>
<br>
<br>
<h2>LRU 캐시와 같은 특정한 자료구조를 사용한다.</h2><p>캐시가 가득 찼을 때 가장 오랫동안 참조되지 않은 객체를 제거하는 자료구조로서 메모리 누수를 방지하기 위해 해당 자료구조를 사용할 수 있다.</p><br>
<br>
<br>
<h1>정리</h1><ul>
<li>직접 메모리를 관리하는 경우 (Stack, Cache, Listener 등) 메모리 누수를 주의하자</li>
<li>여러가지 대안이 있으나 필요하지 않은 객체를 <code>null</code>처리 하거나 백그라운드 스레드를 통해 객체를 정리할 수 있다.</li>
</ul><br>
<br>
<br>
<br>
<p>원본 코드 출처: 이펙티브 자바<br>
참조 및 코드 출처: 이펙티브 자바 완벽 공략 1부 - 백기선, 인프런</p>