## Rule6. 유효기간이 지난 객체 참조는 폐기하라.
//author : choijaesun1
### 1. 메모리 누수
```JAVA
(예시) 메모리 누수가 어디서 생길까?
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public Object pop() {
    if(size == 0)
      throw new EmptyStackException();
    return elements[--size];
  }

  /**
  * 적어도 하나 이상의 원소를 담을 공간을 보장한다.
  * 배열의 길이를 늘려야 할 때마다 대략 두 배씩 늘인다.
  */
  private void ensureCapacity() {
    if(elements.length == size)
      elements = Array.copyOf(elements, 2 * size + 1);
  }
}
```
- 스택이 커졌다가 줄어들면서 객체의 참조가 해제되지 않아서 GC가 해당 객체를 수집하지 못한다.

- elements배열에서 실제로 사용되는 부분을 제외한 나머지 영역을 만기참조영역(다시 이용되지 않을 Reference)이라고 하는데 스택이 만기 참조를 제거하지 않기 때문

- 자바에서 한 객체가 누수될 경우, 그 객체뿐만 아니라 그 객체가 참조하고 있는 모든 객체가 같이 누수됨.

### 2. 해결 방법
- 쓸 일 없는 객체 참조는 무조건 null로 만든다.
```JAVA
public Object pop(){
  if(size == 0)
    throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null;
  return result;
}
```
- 그렇다고 객체사용이 끝나면 모두 null 처리 하지 않아도 됨. 규범이라기 보단 예외적인 조치

- 예외적인 경우 == 자체적으로 관리하는 메모리가 있는 클래스를 만들 경우.

- 가장 좋은 밥업은 변수를 정의할 때 그 유효범위를 최대한 좁게 만들어 객체가 자연스럽게 유효범위를 벗어나도록 설계

- 캐시도 메모리 누수를 조심하자!

- 객체 Reference를 캐시 안에 넣어 놓고 잊어버리는 일이 많다.

- WeakHashMap 를 활용하자.

 - 캐시 바깥에서 키를 참조하고있을 때만 Value를 보관.
 - 키에대한 참조가 만기참조가 되는 순간 캐시안에 보관된 키-값은 자동으로 삭제


- background Thread를 이용해서 캐시에서 사용하지 항목들을 처리한다.

- 새 항목을 캐시에 추가할 때 확인하여 처리 (LinkedHasMap을 사용하면 구현하기 좋다고 함. 왜죠..?)

WeakReference : GC 대상

Listener 등의 Callback 에서도 메모리 누수 발생 가능!

- callback을 명시적으로 제거하지 않는다면 메모리가 점유된 상태로 남음. 누적.
- GC가 callback을 즉시 처리할 수 있도록 weak Reference만 저장하도록 한다. ..? 뭔소리지 예제좀?
