<h1>11. equals를 재정의하려거든 hashCode도 재정의하라.</h1>
<br>
<br>
<br>
<h2>개요</h2><p><code>equals</code>와 <code>hashCode</code>는 항상 같이 구현되어야 한다. IDE, Lombok, AutoValue 등 서도 같이 구현되는 것을 전제로 한다.</p>

<br>
<br>
<br>
<h2>일반 규약</h2><ul>
<li><code>equals</code> 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야 한다. (변경되거나, 애플리케이션을 다시 실행했다면 달라질 수 있다.)</li>
<li><code>equals</code>가 두 객체를 같다고 판단했다면, <code>hashCode</code>의 값도 같아야 한다.</li>
<li>두 객체에 대한 <code>equals</code>가 다르더라도, <code>hashCode</code>의 값은 같을 수 있지만 <b>해시 테이블 성능을 고려해 다른 값을 리턴하는 것이 좋다.</b></li>
</ul>
<br>
<br>
<br>

<h3>제약사항</h3><ol>
<li><code>equals</code>에서 사용한 필드들을 모두 사용해야 한다.이렇게 해야 인스턴스 간의 차이점이 <code>hashCode</code>에 반영되어 다른 인스턴스가 같은 해쉬코드를 가지는 것해쉬 충돌(hash collision) 을 최대한 방지할 수 있다.</li>
</ol>
<p>구체적으로는, <code>HashMap</code>에 put을 하면 Key의 <code>hashCode</code> 값으로 <code>hashBucket</code>에 집어 넣는다. <code>hashBucket</code>안에는 <code>LinkedList</code>가 구현되어 있다. 따라서 <code>hashCode</code>의 값이 동일한 hash collision이 일어나면 탐색의 시간 복잡도가 O(n)이 되는 것이다.</p>
<br>
<br>
<br>


<h2>구현 방법</h2><ol>
<li>첫 번째 핵심필드로 hashCode 값을 구하고 <code>int result</code> 값에 대입한다.</li>
<li>그 다음 부터 정해진 순서대로 핵심필드들의 hashCode 값을 구해서 기존의 값에 31을 곱한 것에 더한다.EX) <code>result = 31 * result + Short.hashCode(shortVariable)</code>
<ol>
<li>레퍼런스 타입은 그 타입의 <code>hashCode</code>메서드를 호출한다.</li>
<li>Primitive 타입은 <code>Type.hashCode(var)</code>을 호출하면 된다.</li>
<li>Array(배열)은 <code>Arrays.hashCode()</code>를 호출하면 된다.</li>
</ol>
</li>
<li><code>result</code>를 반환한다.</li>
</ol>


<p>아래는 예시코드이다.</p><pre><code> // 코드 11-2 전형적인 hashCode 메서드 (70쪽)
@Override 
public int hashCode() {
    int result = Short.hashCode(areaCode); // 1
    result = 31 * result + Short.hashCode(prefix); // 2
    result = 31 * result + Short.hashCode(lineNum); // 3
    return result;
}
</code></pre>
<br>
<br>
<br>
<h3>31인 이유?</h3><ol>
<li>홀수짝수의 경우 0이 채워지면서 정보가 손실될 수 있다.(컴퓨터에서 변수의 표현이 내부적으로 2진수 이므로)</li>
<li>어떤 개발자가 사전에서 해싱을 시도했는데 31이 가장 해쉬 충돌이 적게 일어났다고 한다.</li>
</ol>
<br>
<br>
<br>
<h2>실질적인 구현</h2><p><code>Object.hash()</code>를 사용하는 것이 간편하다. (내부적인 구현 방식도 위 코드와 비슷하다)</p><pre><code> // 코드 11-3 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다. (71쪽)
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
</code></pre><p>하지만 성능에 민감한 경우 성능이 약간 저하될 수도 있다는 점은 감안하자. (대부분의 경우는 상관 없다.)</p>
<ul>
<li>불변 객체의 경우 미리 <code>hashCode</code>를 캐싱해 놓을 수 있다.</li>
<li><code>hashCode</code>를 지연 초기화할 경우 쓰레드 안정성에 신경써야 한다.</li>
</ul>

<br>
<br>
<br>
<h2>정리</h2><ul>
<li><code>equals</code>를 쓰려면 <code>hashCode</code>도 반드시 재정의하자.</li>
<li><code>equals</code>에서 사용된 핵심 필드들을 <code>hashCode</code>를 계산할 때 모두 사용하자.</li>
<li>간편하게 Lombok의 <code>@EqualsAndHashCode</code>를 사용하자.&rarr; Lombok을 이용할 경우 단위 테스트를 할 필요 없다는 점도 장점이다.</li>
</ul>