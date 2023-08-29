<h1>6. 불필요한 객체 생성을 피하라</h1><br>
<h2>불필요한 객체 생성</h2><p><code>String</code> 타입의 인스턴스의 경우 한 번 생성되면 <b>JVM 내부의 메모리에 캐싱</b>되기 때문에 같은 JVM내에 있다면 <b>모든 코드가 동일한 인스턴스를 참조</b>한다. 하지만,  <code>new String("문자열")</code>과 같은 생성자를 호출한다면 새로운 인스턴스가 생성된다. 반복문 내에 있으면 <b>불필요한 객체가 수백만개 생성될 수 있으니 주의</b>할 필요가 있다.</p><br>
<br>
<br>
<h2>생성 비용 절약</h2><p>책에서는 생성 비용이 아주 비싼 객체들은 캐싱해서 재사용하기를 권장한다.<br>
예를 들어서, <code>String.matches</code> 메서드의 경우 패턴을 입력 받으면 내부적으로 패턴을 컴파일해서 인스턴스를 만들게 되는데 생성 비용이 높다. 따라서, <b>동일한 패턴이 자주 사용되는 경우라면 필드로 선언해서 캐싱해놓고 재사용하는 것이 바람직</b>하다.</p><pre><code>// String.matches는 매번 입력된 패턴을 컴파일해서 인스턴스를 생성하므로
// 비용이 아주 비싸다.
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

// 따라서 자주 사용되는 패턴은 아래처럼 캐싱해놓고 사용하는 것이 성능 향상에
// 도움이 될 것이다.
private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
</code></pre><br>
<br>
<br>
<h2>오토박싱</h2><p>primitive type을 reference type 으로 변환할 때 오토박싱이 일어난다. 반대로 reference type을 primitive type으로 바꾸는 것을 언박싱이라고 한다.</p><br>
<p>문제는 <b>오토박싱이 반복문을 통해서 수 없이 반복</b>되면 <b>인스턴스가 계속해서 만들어지기 때문에 성능이 급격히 하락</b>한다. 따라서 의미 없는 오토박싱이 반복되지 않도록 주의해야 한다.</p><pre><code>private static long sum() {
    // Long (레퍼런스 타입으로 선언)
	Long sum = 0L;
	for (long i = 0; i &lt;= Integer.MAX_VALUE; i++)
		// i가 Long 으로 오토박싱된다.
		// 이 과정에서 수 많은 인스턴스가 생성된다.
		sum += i;
	return sum;
}
</code></pre><br>
<br>
<br>
<br>
<br>
<p>원본 코드 출처: 이펙티브 자바<br>
참조 및 코드 출처: 이펙티브 자바 완벽 공략 1부 - 백기선, 인프런</p>