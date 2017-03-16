## Rule26. 가능하면 제네릭 자료형으로 만들 것
제네릭 자료형을 직접 만드는 것은 까다롭지만, 배울 가치가 있다.
스택(stack) (규칙6)을 예시로 하여 제네릭 자료형을 직접 구현해보자.

```JAVA
// Object를 사용한 컬렉션 - 제네릭을 적용할 중요 후보
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public Object pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; // 만기 참조 제거
    return result;
  }

  public boolean isEmpty() {
    return size == 0;
  }

  private void ensureCapacity() {
    if (elements.length == size) {
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
```

클래스를 제네릭화 하는 첫 번째 단계는 선언부에 __형인자(type parameter)__ 를 추가하는 것이다.
그리고 Object를 자료형으로 사용하는 부분들을 전부 찾아서 형인자(E)로 대체하고 컴파일해보자.

```JAVA
// 제네릭을 사용해 작성한 최초 Stack 클래스 - 컴파일되지 않는다!
public class Stack {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public stack() {
    elements = new E[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public E pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    E result = elements[--size];
    elements[size] = null; // 만기 참조 제거
    return result;
  }

```

```JAVA
Stack.java:8: generic array creation
  elements = new E[DEFAULT_INITIAL_CAPACITY]
```

E[DEFAULT_INITIAL_CAPACITY] 객체를 생성하는 과정에서 에러가 난다.

규칙25 에서 설명한 대로, E 같은 실체화 불가능 자료형은 배열을 생성할 수 없다.  
두 가지 해결책이 있는데,  

1. Object의 배열을 만들어서 제네릭 배열 자료형으로 형 변환(cast)하는 방법이다.
 - 문법적으로 문제는 없지만, 일반적으로 형 안전성을 보장하는 방법은 아니다.
 - 무점검 형변환(unchecked cast)을 하기 전에 개발자는 반드시 그런 형변환이 프로그램의 형 안전성을 해치지 않음을 확실히 해야 한다.
 - 배열 elements는 private 필드이고, 클라이언트에 반환되지 않으며 어떤 메서드에도 전달되지 않으므로 문제가 없다.
 - 경고를 억제해도 무방하다.

```JAVA
//elements 배열에는 push(E)를 통해 전달된 E 형의 객체만 저장된다.
//이 정도면 형 안전성은 보장할 수 있지만, 배열의 실행시간 자료형은 E[]가
//아니라 항상 Object[]이다!
@SuppressWarning("unchecked")
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

2. elements의 자료형을 E[]에서 Object[]로 바꾸는 것이다.

``` JAVA
Stack.java:19: incompatible types
found     : Object, required : E
          E result = elements[--size];
                             ^
```
다음과 같은 오류 메시지를 보게 된다. 배열에서 꺼낸 원소의 자료형을 Object에서 E로 변환하도록 코드를 수정하면,
오류는 경고로 바뀐다.

```JAVA
//무점검 경고를 적절히 억제한 사례
public E pop() {
  if (size == 0) throw new EmptyStackException();

  //자료형이 E인 원소만 push 하므로, 아래의 형변환은 안전하다.
  @SuppressWarning("unchecked")
  E result = (E) elements[--size];

  elements[size] = null; //만기 참조 제거
  return result;
}
```

3. 제네릭 Stack 사용 예
```JAVA
public static void main(String[] args) {
  Stack<String> stack = new Stack<String>();
  for(String arg : args) {
    stack.push(arg);
  }
  while(!stack.isEmpty()) {
    System.out.println(stack.pop().toUpperCase());
  }
}
```
