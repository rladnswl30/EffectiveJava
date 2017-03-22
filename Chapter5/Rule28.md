## Rule28. 한정적 와일드카드를 써서 API 유연성을 높여라
### 불변 자료형보다 높은 유연성을 필요할 때는 와일드카드 자료형을 써라.
- 형인자 자료형(parameterized type)은 불변 자료형이다.([규칙25](/Chapter5/Rule25.md))
- 불변 자료형보다 높은 유연성을 필요로 할 때가 있다.

```JAVA
// Stack Class
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```

```JAVA
//와일드카드 자료형을 사용하지 않는 pushAll 메서드 - 문제가 있음!
public void pushAll(Iterable<E> src) {
  for (E e : src) {
    push(e);
  }
}
```
Iterable src가 가리키는 원소의 자료형이 스택 원소의 자료형과 일치하면 제대로 동작한다.  
하지만, Stack< Number >인 경우 Integer형(Number의 하위 자료형)의 intVal로 push(intVal)을 하면 오류 메시지가 출력된다.  
-> 형인자 자료형은 불변이기 때문!

- 해결법 : 한정적 와일드카드 자료형(bounded wildecard type)를 써서 상위-하위 자료형에 있어서 유연성 제공

```JAVA
// E 객체 생산자 역할을 하는 인자에 대한 와일드카드 자료형
//"E의 하위자료형의 Iterable"
public void pushAll(Iterable<? extends E> src) {
  for(E e : src) {
    push(e);
  }
}
```

<br>

```JAVA
//와일드카드 자료형 없이 구현한 popAll 메서드 - 문제 있음!
public void popAll(Collection<E> dst) {
  while(!isEmpty()) {
    dst.add(pop());
  }
}
```

Stack< Number >이고 Object 형의 변수가 하나 있을 때, Collection< Object >가 Collection< Number >의 하위자료형이 아니라는 오류메시지 출력.

- 해결법 : 한정적 와일드카드 자료형(bounded wildecard type)

```JAVA
//E의 소비자 구실을 하는 인자에 대한 와일드카드 자료형
//"E의 상위자료형의 Collection"
public void popAll(Collection<? super E> dst) {
  while(!isEmpty()) {
    dst.add(pop());
  }
}
```

즉, 유연성을 최대화 하려면, 객체 생산자(producer)나 소비자(consumer) 구실을 하는 메서드 인자의 자료형은 __와일드카드 자료형__ 으로 하자!

```
와일드카드 자료형 사용시 지켜야 할 기본적 원칙 (Get and Put Principle. by 나프탈린과 와들러)
PECS (Produce - Extends, Consumer - Super)
T가 생산자라면 <? extends T>
T가 소비자라면 <? super T>
```

### 반환 값에는 와일드카드 자료형을 쓰면 안된다.
```JAVA
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```
반환값 자료형은 여전히 Set< E >.  
반환값에 와일드카드 자료형을 쓰면, 클라이언트 코드 안에도 와일드카드 자료형 명시해야 함!

### 명시적 형인자(explicit type parameter)의 사용
```JAVA
Set<Integer> integer = ...;
Set<Double> doubles = ...;
Set<Number> numbers = union(integers, doubles);
```
컴파일러가 자료형을 정확히 유치하지 못하여 오류 메시지가 뜬다.   
-> 명시적 형인자(explicit type parameter)로 해결

```JAVA
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

### PECS 원칙을 적용한 Comparable, Comparator
```JAVA
//max 메서드(규칙 27)
public static <T extends Comparable<T>> T max(List<T> list)
```

```JAVA
//와일드카드 자료형 사용
public static<T extends Comparable<? super T>> T max(List<? extends T> list)
```
형인자 T에 PECS 원칙 적용.  
Comparable은 언제나 소비자이므로, Comparable< T > 대신 Comparable<? super T> 를 사용해라!  
Comparator< T >도 마찬가지로 Comparator<? super T>를 사용하자.

### 형인자와 와일드카드 사이에 존재하는 이원성(duality) 문제
```JAVA
//swap 메서드 선언의 두가지 방법
public static<E> void swap(List<E> list, int i, int j); //1
public static void swap(List<?> list, int i, int j); //2
```
무엇이 더 바람직할까?  

2번이 더 바람직. 더 간단하기 때문.  
- 메서드에 아무 리스트나 인자로 전달하면 첨자가 가리키는 원소들을 바꿔준다.
- __형인자가 메서드 선언에 단 한군데 나타난다면 해당 인자를 와일드카드로 바꿔라.__

```JAVA
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```
리스트에서 방금 꺼낸 원소를 그 리스트에 다시 넣을 수 없는 문제점!  
list<?>에는 null 이외의 어떤 값도 넣을 수 없다.

```JAVA
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

//와일드카드 자료형을 포착하기 위한 private 도움 메서드
public static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```
list가 List< E >라는 것을 안다. 따라서 해당 리스트에서 꺼낸 값의 자료형은 E이다.  
E형의 값은 리스트에 넣어도 안전하다.

### 요약
- API에는 와일드카드 자료형을 써서 유연성을 높여라
- 생산자는 extends, 소비자는 super라는 PECS 규칙을 외우자
- 모든 Comparable과 Comparator는 소비자다
