<h1>12. toString을 항상 재정의하라.</h1>
<br>
<br>
<h2>기본 구현</h2><p><code>Object</code>에서 기본적으로 제공하는 <code>toString</code>은 <code>클래스명 + 16진수hashCode</code>값을 반환한다. &larr; 그닥 유용하지 않다.</p>
<br>
<br>
<h2>재정의</h2><p>해당 데이터에 적합한 형태로 표현해주는 것이 좋다. <code>toString</code>에 들어가는 필드(혹은 정보)는 외부에 공개할 수 있는 데이터만 공개하는 것이 좋다. 외부에 노출 할 수없는 정보(사용자 정보)는 최대한 숨기자.</p>
<ul>
<li><code>Getter</code>를 제공하여 <code>toString</code>에 나오는 정보를 접근 가능하게 하자. 어차피 해당 정보를 얻기 위해서 클라이언트가 정해진 포맷에 따라 <code>toString</code>을 파싱할 것이다.</li>
<li><b>IDE나 Lombok을 사용할 수도 있지만 자동으로 생성되는 포맷이 원하는 포맷이 아닐 수도 있다. 그럴 때는 직접 정의하자.</b></li>
<li>아래와 같이 정적 팩토리 메서드로 정해진 포맷이 입력되면 파싱하여 객체를 만들 수도 있다.</li>
</ul><pre><code>public static PhoneNumber of(String phoneNumberString) {
    String[] split = phoneNumberString.split("-");
    PhoneNumber phoneNumber = new PhoneNumber(
            Short.parseShort(split[0]),
            Short.parseShort(split[1]),
            Short.parseShort(split[2]));
    return phoneNumber;
}
</code></pre>