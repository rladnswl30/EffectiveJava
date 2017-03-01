## Rule18. 추상 클래스 대신 인터페이스를 사용하라
 
### 여러가지 구현을 허용하는 자료형을 만드는 방법 두 가지
 - 추상 클래스(abstract class)
   - 구현된 메서드를 포함
   - 추상 클래스가 규정하는 자료형을 구현하기 위해서는 추상 클래스를 반드시 __계승__ 해야 한다.

 - 인터페이스(interface)
   - 구현된 메서드를 포함 X
   - 인터페이스에 포함된 모든 메서드를 정의하고, 인터페이스가 규정하는 일반 규약을 지키면 된다.
   - 클래스가 __클래스 계층에 속할 필요가 없다.__

- 자바는 다중 상속(multiple inheritance)을 허용하지 않기 때문에, 추상 클래스를 사용하면 자료형 사용에 있어서 많은 제약 발생!

### 이미 있는 클래스를 개조해서 새로운 인터페이스를 구현하도록 하는 것은 간단하다
- 필요한 메서드 추가 후, 클래스 선언부에 implements 넣기

### 인터페이스는 믹스인(mixin)을 정의하는 데 이상적이다
- 믹스인은 클래스가 "주 자료형(primary type)" 이외에 추가로 구현할 수 있는 자료형.  
어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.
- ex) Comparable class
- 즉, 자료형의 주된 기능 + 선택적 기능을 "혼합(mix in)"
- 추상 클래스는 믹스인 사용 불가  
-> 클래스가 가질 수 있는 상위 클래스가 하나 뿐이며, 클래스 계층에는 믹스인을 넣기 좋은 곳이 없다.

### 인터페이스는 비 계층적인 자료형 프레임워크를 만들 수 있도록 한다.
- 어떤 것들은 자료형 계층으로 잘 정리되어 있지만, 계층 안에 잘 들어맞지 않는 것들도 있다.

ex) 가수와 작곡가를 표현하는 인터페이스
```JAVA
public interface Singer {
  AudioClip sing(Song s);
}

public interface Songwriter {
  Song compose(boolean hit);
}
```

ex) 가수이면서 작곡가인 사람을 표현하는 인터페이스
```JAVA
public interface SingerSongwriter extends Singer, Songwriter {
  AudioClip strum();
  void actSensitive();
}
```

- 이러한 인터페이스를 쓰지 않으려면 속성 조합마다 별도의 클래스를 만들어 거대한 클래스 계층을 만들어야 한다.  
-> 조합폭증(combinatiorial explosion) : 필요한 속성이 N개면 지원해야 하는 조합 가짓수는 2^n개

### 인터페이스를 사용하면 포장 클래스 숙어(wrapper class idiom)을 통해 안전하면서도 강력한 기능 개선이 가능하다 __([규칙16](/Chapter4/Rule16.md))__
- 추상 클래스를 사용해 자료형을 정의하면 계승 이외의 수단을 사용할 수 없고, 강력하지 않으며, 깨지기 쉽다.

### 추상 골격 구현(abstract skeletal implementation) 클래스를 중요 인터페이스마다 두면, 인터페이스의 장점과 추상 클래스의 장점을 결합할 수 있다
- 인터페이스로는 자료형을 정의하고, 골격 구현 클래스로 구현
- 관습적으로 골격 구현 클래스의 이름은 __AbstractInterface__ 로 정한다.
- ex) Collection Framework에는 인터페이스별로 골격 구현 클래스들이 하나씩 제공.  
AbstractColletion, AbstractSet, AbstractList, AbstractMap

- 골격 구현의 강력함을 잘 보여주는 예제
```JAVA
//골격 구현 위에서 만들어진 완전한 List 구현
static List<Integer> intArrayAsList(final int[] a) {
    if (a == null)
      throw new NullPointerException();

    //골격 구현 List 반환  
    return new AbstractList<Integer>() {
      public Integer get(int i) {
        return a[i];  // 자동 객체화
      }

      @Override
      public Integer set(int i, Integer val) {
        int oldVal = a[i];
        a[i] = val;     //자동 비객체화
        return oldVal;  //자동 객체화
      }

      public int size() {
        return a.length;
      }
    };
}
```
- int 배열을 Integer 객체의 리스트처럼 볼 수 있도록 하는 어댑터 적용 사례 __([규칙5](/Chapter2/Rule5.md))__
- 정적 팩터리 메서드를 통해 반환되는 객체의 클래스가 정적 팩터리 안에 숨겨진, 외부에서 접근 불가능한 익명 클래스(anonymouse class)
- 추상 클래스를 자료형 정의 수단으로 사용했을 때 만족해야 하는 심각한 제약사항을 따르지 않아도 됨
- 골격 구현 클래스를 상속하도록 기존 클래스를 변경할 수 없다면, 인터페이스를 직접 구현해도 됨  
-> 골격 구현 클래스를 계승하는 private 내부 클래스(inerr class)를 정의하고, 인터페이스 메서드에 대한 호출은 해당 중첩 클래스 객체로 전달. ( 모의 가상 상속(simulated multiple inheritance) __[규칙16](/Chapter4/Rule16.md)__ )
- 골격 구현 클래스를 만들 때, 우선 인터페이스를 살펴보고, 다른 메서드를 구현할 때 쓸 기본 메서드(primitive method)들을 추려낸다. 이 기본 메서드들은 골격 구현 클래스에서 추상 메서드로 선언한고, 나머지 메서드들은 실제로 구현해서 넣는다.
- 골격 구현 클래스 예시
```JAVA
//골격 구현
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V
 {
      //기본 메서드(primitive method)
      public abstract K getKey();
      public abstract V getValue();

      //변경 가능 맵에 들어갈 Entry는 이 메서드에 재정의
      public V setValue(V value) {
        throw new UnsupportedOperationException();
      }

      //Map.Entry.equals의 일반 규약 구현
      @Override
      public boolean equals(Object o) {
        ...
      }
      private static boolean equals(Object o1, Object o2) {
        ...
      }

      //Map.Entry.hashCode의 일반 규약 구현
      @Override
      public int hashCode() {
        ...
      }
      private static int hashCodE(Object obj) {
        ...
      }
}
```

- 골격 구현 클래스는 계승 용도로 설계하는 클래스이므로, __[규칙17](/Chapter4/Rule17.md)__ 의 모든 지침을 따라야 한다.
- 문서화를 제대로 하는 것도 중요
- 간단 구현(simple implementation) : 골격 구현의 작은 변종. 인터페이스를 구현한 클래스로 계승을 고려해 만들어진 클래스라는 점에서는 골격 구현과 같지만 추상 클래스가 아니다.  
ex) AbstractMap.SimpleEntry

### 인터페이스보다는 추상 클래스가 발전시키기 쉽다
- 다음 릴리즈에 새로운 메서드를 추가하고 싶다면, 적당한 기본 구현 코드를 담은 메서드를 언제든 추가할 수 있다.  
-> 해당 추상 클래스를 계승하는 모든 클래스는 즉시 새로운 메서드를 제공
- 일반적으로, public 인터페이스를 구현하는 기존 클래스를 깨뜨리지 않고 새로운 메서드를 인터페이스에 추가할 방법은 없다. (java 1.8부터는 가능)

### public 인터페이스는 신중히 설계하자
- 인터페이스가 공개되고 널리 구현된 다음에는, 인터페이스 수정이 거의 불가능
- 결함이 심각하면 API 전체를 못쓰게 될 수도 있으니 인터페이스를 다양한 방법으로 구현해보도록 하자

### 요약
- 인터페이스는 다양한 구현이 가능한 자료형을 정의하는 일반적으로 가장 좋은 방법
- 개선이 쉬운 API를 만들 때에는 추상 클래스가 좋다
- 중요한 인터페이스를 API에 포함시키는 경우, 골격 구현 클래스를 함께 제공하면 어떨지 고려하라
- public 인터페이스 설계 시 극도로 주의하자
