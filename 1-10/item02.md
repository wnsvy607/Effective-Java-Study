<h1>아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라.</h1><br>
<br>
<h2>점층적 생성자 패턴</h2><p>정적 팩토리, 생성자 모두 선택적 매개 변수가 많을 경우, 이를 핸들링 하기위해서 프로그래머들은 전통적으로 <b>점층적 생성자 패턴(telescoping constructor pattern)</b>을 사용해왔다.</p><br>
<pre><code>public NutritionFacts(int servingSize, int servings,
                      int calories, int fat, int sodium, int carbohydrate) {
    this.servingSize  = servingSize;
    this.servings     = servings;
    this.calories     = calories;
    this.fat          = fat;
    this.sodium       = sodium;
    this.carbohydrate = carbohydrate;
}

public NutritionFacts(int servingSize, int servings) {
    this(servingSize, servings, 0);
}

public NutritionFacts(int servingSize, int servings,
                      int calories) {
    this(servingSize, servings, calories, 0);
}

public NutritionFacts(int servingSize, int servings,
                      int calories, int fat) {
    this(servingSize, servings, calories, fat, 0);
}

public NutritionFacts(int servingSize, int servings,
                      int calories, int fat, int sodium) {
    this(servingSize, servings, calories, fat, sodium, 0);
}

</code></pre><br>
<p>위의 코드와 같이 매개변수가 적은 쪽에서 많은 쪽으로 생성자를 호출하여 구성하는 방법이다.</p><br>
<br>
<br>
<h3>단점</h3><p>매개변수가 늘어날 수록 클라이언트 코드를 작성할 때 프로그래머가 <b>매개변수의 개수와 순서에 항상 유의</b>해야 하며 코드를 읽기 어렵게 만든다.</p><br>
<br>
<br>
<br>
<h2>자바 빈즈 패턴</h2><p>자바 빈즈 패턴이란 매개 변수가 없는 생성자를 통해 인스턴스를 우선 생성한 뒤, 자바빈즈 규약에 의해 정의된 setter로 내부의 값을 지정하는 방법이다.</p><pre><code>public void setServingSize(int servingSize) {
    this.servingSize = servingSize;
}

public void setServings(int servings) {
    this.servings = servings;
}

public void setCalories(int calories) {
    this.calories = calories;
}

public void setFat(int fat) {
    this.fat = fat;
}

public void setSodium(int sodium) {
    this.sodium = sodium;
}

public void setCarbohydrate(int carbohydrate) {
    this.carbohydrate = carbohydrate;
}
</code></pre><br>
<h3>단점</h3><ol>
<li>이 방법은 심각한 단점을 지니는데, 객체가 완성되기 전 까지<b>일관성(Consistency)이  무너진 상태에 놓여 있다는 것이다.</b> 일관성이 깨진 객체가 다른 코드에서 호출(사용)된다면 버그를 심은 코드와 버그가 발생한 코드가 물리적으로 떨어져 있을 확률이 높기 때문에 디버깅을 더욱 더 어렵게 만든다. 또한, 어느 시점에 일관성이 보장되는 지도 알 수가 없다.(문서화가 필요함)<br></li>
<li>자바빈즈 규약을 따름으로써 객체를 불변으로 만들 수 없다는 것도 단점이다. 이를 방지하기 위해 객체 freezing이라는 기법을 사용할 수 있지만 개발자로 하여금 고민할 포인트를 더 만들기 때문에 현업에서 잘 사용되지 않는다.(비실용적)<br></li>
<li>객체 하나를 만들기 위해서 수 많은 메서드(setter)를 호출해야 한다.</li>
</ol><br>
<h2>빌더 패턴</h2><p>두 방법의 단점을 적절히 해결해 줄 수 있는 대안이 있는데, 바로 빌더 패턴이다.</p><pre><code>public static class Builder {
    // 필수 매개변수
    private final int servingSize;
    private final int servings;

    // 선택 매개변수 - 기본값으로 초기화한다.
    private int calories      = 0;
    private int fat           = 0;
    private int sodium        = 0;
    private int carbohydrate  = 0;

    public Builder(int servingSize, int servings) {
        this.servingSize = servingSize;
        this.servings    = servings;
    }

    public Builder calories(int val)
    { calories = val;      return this; }
    public Builder fat(int val)
    { fat = val;           return this; }
    public Builder sodium(int val)
    { sodium = val;        return this; }
    public Builder carbohydrate(int val)
    { carbohydrate = val;  return this; }

    public NutritionFacts build() {
        return new NutritionFacts(this);
    }
}

private NutritionFacts(Builder builder) {
    servingSize  = builder.servingSize;
    servings     = builder.servings;
    calories     = builder.calories;
    fat          = builder.fat;
    sodium       = builder.sodium;
    carbohydrate = builder.carbohydrate;
}
</code></pre><br>
<p>객체 생성만을 담당하는 Builder 정적 클래스를 따로 만들어 주는 것이다. Builder에 생성자에 <b>필수 매개변수를 설정할</b> 수 있으며, <b>선택 매개변수는 Builder 클래스 내의 메서드를</b> 통해 설정할 수 있다. 이 때, Builder 그 자신을 return함 으로써 <b>메서드 체이닝(혹은 Fluent API)</b>을 이용하는 것도 키 포인트다.</p><br>
<p>빌더 패턴은 파이썬과 스칼라에 있는 명명된 선택적 매개변수(named optional parameters)를 흉내낸 것이다.</p><br>
<p>책에서는 여기서 매개변수의 유효성 검사를 생략했으나 실제로 작성할 때는 유효성 검사를 작성할 것을 권장하고 있다.</p><br>
<h2>장점</h2><ol>
<li>객체의 <b>일관성</b>을 지킬 수 있다.(안전성) 객체가 명시적으로 완성될 때에야 비로소 build() 메서드를 호출하여 새 인스턴스를 얻으며 Builder 자체만으로는 객체 생성만 가능하기 때문에 일관성 문제에서 자유로워진다. (자바 빈즈 패턴)<br></li>
<li>클라이언트 입장에서 코드를 읽고 쓰기가 수월해진다. 메서드의 이름을 통해서 어떤 선택적 매개변수인지 의도가 명확하게 드러나기 때문이다.(점층적 생성자 패턴)<br></li>
<li>가변인수를 사용할 수 있다.(생성자)<br></li>
<li>계층적 빌더구조를 만들 수 있다.</li>
</ol><br>
<p>abstract class를 만들고 공변환 타이핑(covariant return typing) 기능을 사용하여 abstract class를 상속한 class도 형변환 없이 Builder를 사용할 수 있다.<br>
&rarr; 빌더를 직접 작성해야 하기 때문에 잘 사용하지 않을듯 하다.</p><br>
<h2>단점</h2><ul>
<li>코드가 장황하고 작성하기가 쉽지 않다.</li>
</ul><br>
<p>&rarr; Lombok 라이브러리를 통해서 <code>@Builder</code> 어노테이션만으로 간단하게 빌더를 구현할 수 있다.<br>
그러나</p><ul>
<li>기본으로 모든 매개변수를 받는 생성자가 생성된다.<br>&rarr; <code>@AllArgsConstructor(access = AccessLevel.PRIVATE)</code>으로 극복 가능하다.<br></li>
<li>필수 매개변수를 지정할 수가 없다는 단점이 있다.<br></li>
<li>성능이 매우 민감한 경우 성능 저하를 유발할 수 있다.</li>
</ul><br>
<br>
<h2>결론</h2><ul>
<li>매개 변수가 4개 이상일 경우에는 빌더 패턴을 사용하자.</li>
<li>API는 시간이 지나면서 매개 변수가 많아지는 경향이 있으므로 처음 설계부터 빌더 패턴을 고려해보자.</li>
<li>매개 변수중 다수가 필수가 아닌 선택이고 같은 타입이 많을 경우 빌더 패턴의 가치는 더욱 더 상승한다.</li>
</ul><br>
<h2>강의</h2><p>Builder 패턴은</p><ol>
<li>코드를 작성하기 쉽지 않으며, 이해하기 어렵다.</li>
<li>필드의 중복이 일어난다.</li>
</ol><br>
<p>따라서,</p><ol>
<li>클래스 생성에 필수적인 매개변수와, 선택적인 매개변수가 너무 많고 이것 때문에 생성자의 매개 변수가 너무 많이 늘어난다.</li>
<li>Immutable하게 만들고 싶다.</li>
</ol><br>
<p>두 가지 경우에 Builder를 사용할 것을 권장한다.</p><br>
<h3>빌더 패턴의 목적</h3><p>복잡한 객체를 만드는 프로세스를 독립적으로 분리할 수 있다. (SRP)</p><br>
