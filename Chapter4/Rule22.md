## Rule22. 멤버 클래스는 가능하면 static으로 선언하라
- 중첩 클래스(nested class)에는 정적 멤버 클래스(static member class), 비-정적 멤버 클래스(nonstatic member class), 익명 클래스(anonymouse class), 지역 클래스(local class)가 있다.
- 정적 멤버 클래스를 제외하고 모두 내부 클래스(inner class)이다.

### 정적 멤버 클래스(static member class)
- 바깥 클래스의 모든 멤버(private 선언된 것 포함)에 접근할 수 있다.
- 바깥 클래스의 정적 멤버, 다른 정적 멤버와 동일한 접근 권한 규칙을 따름
- 정적 멤버 클래스를 private으로 선언했다면 바깥 클래스만 해당 중첩 클래스에 접근 가능
- 정적 멤버 클래스의 흔한 용례 : 바깥 클래스와 함께 사용할 때만 유용한 public 도움 클래스(helper class)를 정의  
ex) Calculator 클래스의 public static 멤버 클래스인 enum 자료형의 Operation  
-> Calculator.Operation.PLUS, Calculator.Operation.MINUS 로 연산 참조
(규칙 30)

### 비-정적 멤버 클래스(nonstatic member class)
- __바깥 클래스 객체와 자동적으로 연결__ (정적 멤버 클래스와 가장 큰 차이)
- 비-정적 멤버 클래스 안에서는 바깥 클래스의 메서드를 호출할 수도 있고,  
this 한정 구문을 통해 바깥 객체에 대한 참조 획득 가능

```JAVA
class Envelope {
  void x() { System.out.println("Hi"); }
  class Enclosure {
  void x() { Envelope.this.x(); /* 한정됨 */ }
  }
}
```
- 중첩된 클래스의 객체가 바깥 클래스 객체와 독립적으로 존재할 수 있도록 하려면 중첩 클래스는 반드시 정적 멤버 클래스로 선언  
-> __비-정적 멤버 클래스의 객체는 바깥 클래스 객체 없이는 존재할 수 없다__
- 비-정적 멤버 클래스 객체와 바깥 객체와의 연결은 비-정적 멤버 클래스의 객체가 만들어지는 순간에 확립, 그 뒤에는 변경 불가능
- enclosingInstance.new MemberClass(args) 구문을 사용하여 수동적으로 연결할 수 있긴 함
- 어댑터(Adapter) 정의할 때 많이 쓰임
 - Map 인터페이스를 구현하는 클래스들은 비-정적 멤버 클래스를 사용해 컬랙션 뷰(collection view) 구현
 - Set이나 List 같은 컬렉션 인터페이스를 구현하는 클래스는 비-정적 멤버 클래스를 사용해 반복자(Iterator) 구현

```JAVA
//비-정적 멤버 클래스의 전형적 용례
public class MySet<E> extends AbstractSet<E> {
  ...

  public Iterator<E> iterator() {
    return new MyIterator();
  }

  private class MyIterator implements Iterator<E> {
    ...
  }
}
```

- __바깥 클래스 객체에 접근할 필요가 없는 멤버 클래스를 정의할 때, 항상 선언문 앞에 static을 붙여 비-정적 멤버 클래스 대신 정적 멤버 클래스로 만들자__
 - static 생략 시, 모든 객체는 내부적으로 바깥 객체에 대한 참조를 유지하여 시간과 공간 요구량이 늘어나고, 쓰레기 수집(garbage collection)이 힘들어진다.ㅊ
- private 정적 멤버 클래스는 바깥 클래스 객체의 컴포넌트(component)를 표현하는 데에 흔히 쓰인다  
ex) Map 구현 클래스의 Entry 객체 : Entry 객체의 메서드(getKey, getValue, setValue 등)는 맵 객체에 접근할 필요가 없다.
- 멤버 클래스가 API 클래스의 public이나 protected 멤버인 경우, 정적 멤버 클래스로 만들지 비-정적 멤버 클래스로 만들지를 중요하게 고려해보아라

### 익명 클래스(anonymouse class)
- 이름이 없으며, 바깥 클래스의 멤버도 아니다.
- 멤버로 선언하지 않으며, 사용하는 순간에 선언하고 객체를 만든다.
- 비-정적 문맥(nonstatic context) 안에서 사용될 때만 바깥 객체를 갖는다.
- 정적 문맥(static context) 안에서 사용되어도 static 멤버를 가질 수 없다.
- 선언하는 순간에 객체를 만들기 때문에 제약이 많다.
- 함수 객체([규칙21](/Chapter4/Rule21.md))를 정의할 때 널리 쓰인다.
- 프로세스 객체(process object)를 만드는데에도 널리 쓰인다.  
ex) Runnable, Thread, TimerTask
- 정적 팩터리 메서드 안에서도 많이 쓰인다.  
ex) initArrayAsList([규칙18](/Chapter4/Rule18.md))
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

### 지역 클래스(local class)
- 네 종류의 중첩 클래스 가운데 사용 빈도가 가장 낮음
- 지역 변수(local variable)가 선언될 수 있는 곳이라면 어디든 선언 가능
- 지역 변수와 동일한 유효범위 규칙을 따른다
- 멤버 클래스처럼 이름을 가진다
- 반복적으로 사용 가능
- 익명 클래스처럼 비-정적 문맥에서 정의했을 때만 바깥 객체를 갖는다
- static 멤버는 가질 수 없다
- 익명 클래스처럼 가독성을 위해 짧게 작성

### 요약
- 중첩 클래스 네 가지는 각각 어울리는 자리가 다르다.
- 중첩 클래스를 메서드 밖에서 사용할 수 있어야 하거나, 메서드 안에 놓기에 너무 길 때는 __멤버 클래스__ 로 정의해라
- 멤버 클래스의 객체 각각이 바깥 객체에 대한 참조를 가져야 할 경우에는 __비-정적 멤버 클래스__ 로 만들어라. 그렇지 않으면 __정적 멤버 클래스__ 로 만들어라.
- 중첩 클래스가 특정 메서드에 속해야 하고, 오직 한 곳에서만 객체를 생성하고, 해당 중첩 클래스의 특성을 규정하는 자료형이 이미 있다면 __익명 클래스__ 로 만들어라. 그렇지 않으면 __지역 클래스__ 로 만들어라.
