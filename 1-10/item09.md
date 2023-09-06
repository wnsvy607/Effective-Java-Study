<h1>9. try-finally 보다 try-with-resources를 사용하라.</h1><br>
<h2>개요</h2><ul>
<li>자바 7부터 try-finally는 더 이상 최선의 방법이 아니다. (자바 7부터)</li>
<li>try-with-resources를 사용하면 코드가 더 짧고 분명하다.</li>
<li>만들어지는 예외 정보도 더 유용하다.</li>
</ul>


<h2>try-finally의 문제점</h2><pre><code>// 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) &gt;= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
</code></pre><p>위와 같이 구현해야 안전하게 close 할 수 있다.</p>
<p>아래는 잘못 try-finally 구문의 잘못된 자원 회수 방법이다.</p><pre><code>static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	OutputStream out = new FileOutputStream(dst);
	try {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) &gt;= 0)
			out.write(buf, 0, n);


	} finally {
		in.close();
        out.close();

	}
}
</code></pre><p>위 코드의 잘못된 점은 <code>in.close();</code> 에서 예외가 발생하면 <code>out.close()</code>는 호출이 안되서 <code>leak</code>이 발생한다는 것이다.</p>
<p>따라서, 자원이 여러개일 때, <b>try-finally 구문의 중첩구조는 필연적</b>이며 이는 코드의 가독성을 낮추고 작성하게 어렵기 만든다.</p>
<br><br><br>




<h2>try-with-resources</h2><p><b>try-with-resources</b>를 사용하면 JVM이 컴파일 할 때 내부적으로 try-catch 구문을 이용해 안전한 자원회수가 가능하도록 해준다.</p>
<ol>
<li>훨씬 코드를 간결하게 만들어준다.</li>
</ol><pre><code>// 코드 9-4 복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다! (49쪽)
static void copy(String src, String dst) throws IOException {
    try (InputStream   in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) &gt;= 0)
            out.write(buf, 0, n);
    }
}
</code></pre><p>위의 try-finally 구문과 비교하면 훨씬 간결하다.</p>




<ol>
<li>예외를 잡아먹지 않는다.</li>
</ol>
<p>만약, try 구문과 finally 구문 둘 다에서 예외가 발생할 경우 가장 나중에 발생한 예외만 보이게 된다.
예시 코드를 보자.</p><pre><code>//BadBufferedReader.class
@Override
public String readLine() throws IOException {
    throw new CharConversionException();
}

@Override
public void close() throws IOException {
    throw new StreamCorruptedException();
}


//TopLine.class
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
</code></pre><p>위와 같이  <code>readLine</code>, <code>close</code>메서드를 오버라이드 하여 예외를 던지도록 하였다.(단지, 예시를 위해서) 이 경우 <code>readLine</code>에서 먼저 예외가 발생하고 finally 구문으로 넘어가지만 <code>close</code>에서도 예외가 발생하기 때문에, 가장 나중에 발생한 예외인 <code>StreamCorruptedException</code>만 보이게 된다. 문제는 일반적으로 <b>디버깅할 때 가장 처음 발생한 예외에 관심이 있다</b>는 것이다.</p>

<p>하지만, try-with-resources로 다음과 같이 바꿔보자</p><pre><code>static String firstLineOfFile(String path) throws IOException {
    try(BufferedReader br = new BadBufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
</code></pre><p>앞서 말했듯 간결함이라는 장점과 더불어, <b>후속 예외까지 전부 보이게 된다</b>.</p>
<p>결과
<img src='https://github.com/wnsvy607/Effective-Java-Study/assets/85255237/3b8c2e17-8fa9-4aad-b537-ecab87849147'></p>

<br><br>
<h2>결론</h2><ul>
<li>반드시 자원을 정리할 때 try-finally 대신 try-with-resources를 사용하자.</li>
<li>간결함과 후속 예외를 쉽게 확인할 수 있다는 이점을 얻는다.</li>
</ul>
