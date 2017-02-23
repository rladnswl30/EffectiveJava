## Rule17. 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라.
### 1. 재정의 가능 메서드를 내부적으로 어떻게 사용하는지 반드시 문서에 남겨라!
- public이나 protected로 선언된 모든 메서드와 생성자에 대해,  
어떤 재정의 가능 메서드를 어떤 순서로 호출하는지, 그리고 호출 결과가 어떤 영향을 미치는지 문서로 남겨라.
- 재정의 가능 메서드가 호출되는 모든 상황을 문서로 남겨라.  
ex) 후면 스레드 호출, static 초기화 구문(initializer) 안에서 호출
- 주석 예시 : java.util.AbstractCollection
```JAVA
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the collection looking for the
 * specified element.  If it finds the element, it removes the element
 * from the collection using the iterator's remove method.
 *
 * <p>Note that this implementation throws an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by this
 * collection's iterator method does not implement the <tt>remove</tt>
 * method and this collection contains the specified object.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 */
public boolean remove(Object o) {  
      ...
}
```
 - iterator 메서드를 재정의하면 remove가 영향이 받는다는 사실과 iterator 메서드가 반환하는 Iterator 객체가 remove 메서드에 어떤 영향을 주는지 정확히 명시

### 2. 클래스 내부 동작에 개입할 수 있는 훅(hooks)을 신중하게 고른 protected 메서드 형태로 제공해야 한다.
```JAVA
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```
- 이 메서드를 재정의하여 리스트 구현체의 내부 구현을 활용하면, this 리스트와 그 부분 리스트에 대한 clear 연산 성능을 많이 올릴 수 있다.
- protected 멤버 개수는 가능한 줄여라. 구현 세부사항에 대한 일종의 서약 구실을 하기 때문.

### 3. 계승을 위해 설계한 클래스를 테스트할 유일한 방법은 하위 클래스를 직접 만들어 보는 것이다.
- 저자 경험으로 볼 때, 계승에 맞게 클래스를 테스트하는 데는 하위 클래스 세 개면 대체로 충분했다고 함.
- 문서에 명시한 내부 호출 패턴, 메서드와 필드를 protected로 선언  
 -> 함축된 구현 관련 결정들을 영원히 고수해야 한다.
 -> __releas 포함 전에 반드시 하위 클래스를 만들어 테스트 하자!__

### 4. 계승 허용 시 반드시 따라야 할 제약사항
- __생성자는__ 직접적이건 간접적이건 __재정의 가능 메서드를 호출해서는 안된다.__  
<br>
잘못된 예시)
```JAVA
public final class Sub extends Super {
  private final Date date; // 생성자가 초기화하는 final 필드

  Sub() {
    date = new Date();
  }

  //상위 클래스 생성자가 호출하게 되는 재정의 메서드
  @Override
  public void overrideMe() {
    System.out.println(date);
  }

  public static void main(String[] args) {
    Sub sub = new Sub();
    sub.overrideMe();
  }
}
```

- clone이나 readObject 메서드 안에서 직접적이건 간접적이건 재정의 가능한 메서드를 호출하지 않도록 주의하자.

- 계승을 위해 클래스를 설계하면 클래스에 상당한 제약이 가해진다.

- 계승에 맞도록 설계하고 문서화하지 않은 클래스에 대한 하위 클래스는 만들지 않는 것이다.
