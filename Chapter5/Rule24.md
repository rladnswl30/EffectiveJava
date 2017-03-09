## Rule24. 무점검 경고(unchecked warning)를 제거하라

- 제네릭으로 프로그램 했을 때의 컴파일러 경고 :  
무점검 형변환 경고(unchecked cast warning), 무점검 메서드 호출 경고(unchecked method invocation warning), 무점검 제네릭 배열 생성 경고(unchecked generic array creation warning), 무점검 변환 경고(unchecked conversion warning) 등이 있다.

- 무점검 경고는 가능하다면 모두 없애야 좋다.  
-> 형 안전성(typesafe)이 보장되므로

- 제거할 수 없는 경고 메시지는 형 안전성이 확실할 때만 @SupressWarnings("unchecked") 를 사용해 억제하자!
-> 하지만 가능한 작은 범위에서 적용하자. 중요한 경고 메시지를 놓칠수도 있으니...

```JAVA
// @SupressWarnings 적용 범위를 줄이기 위해 지역 변수로!
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    // 아래의 형변환은 배열의 자료형이 인자로 전달되 자료형인 T[]와 같으므로 정확하다
    @SupressWarnings("unchecked") T[] result =
          (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }

  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;

  return a;
}
```
@SupressWarnings("unchecked") 어노테이션을 사용할 때마다, 왜 형 안전성을 위반하지 않는지 밝히는 주석을 반드시 달자!

### 요약
- 무점검 경고는 중요하다. 무시하지 마라.
- 경고 메시지는 최선을 다해 제거해라 -> 형 안전성이 보장된다는 증거니까
- 형 안전성 보장이 입증되면, @SupressWarnings 으로 해당 경고를 억제하되 적용 범위는 최소화해라.
- @SupressWarnings 에 대한 주석을 반드시 달아라.
