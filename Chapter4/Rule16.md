## Rule16. 계승하는 대신 구성하라.
### 1. 계승은 코드 재사용을 돕는 강력한 도구지만, 항상 최선은 아니다.
- 계승을 적절히 사용하지 못한 소프트웨어는 깨지기 쉽다.
- 단일 패키지 안에서 사용하면 안전하다.

### 2. 메서드 호출과 달리, 계승은 캡슐화(encapsulation) 원칙을 위반한다.
- 상위 클래스의 구현은 릴리즈가 거듭되면서 바뀔수 있는데 하위 클래스 코드는 수정된적이 없어도 망가질수 있다.

```JAVA
//계승을 잘못 사용한 사례
class InstrumentedHashSet<E> extends HashSet<E> {
  //요소를 삽입하려 한 횟수
  private int addCount = 0;

  public InstrumentedHashSet() {
  }

  public InstrumentedHashSet(int initCap, float loadFactor) {
    super(initCap, loadFactor);
  }

  @Override
  public boolean add(E e) {
      addCount++;
      return super.add(e);
  }

  @Override
  public boolean addAll(Collection<? extends E> c) {
      addCount += c.size();
      return super.addAll(c);
  }

  public int getAddCount() {
      return addCount;
  }

  public static void main(String[] args) {
      InstrumentedSet<String> s = new InstrumentedSet<String>();
      s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
  }
```
- InstrumentedHashSet은 깨지기 쉬운 클래스!
- 하위클래스 구현을 망가뜨릴 수 있는 요인
 - 메서드 재정의
 - 상위 클래스에 새로운 메서드 추가
   - 어떤 프로그램의 보안이 어떠한 Collection에 추가되는 모든 원소가 특정 술어(predicate)를 만족한다고 할 때,

     그 술어를 계속 만족시키기 위해서는 Collection 하위 클래스에서 원소를 추가할 수 있는 모든 메서드를 재정의해서, Collection에 원소를 넣기 전에 술어가 만족되는지 검사해야 한다.
   - Hashtable과 Vector를 Collection 프레임워크에 넣는 과정에서 보안문제로 수정해야 했다.

### 3. 해결 방법 : 구성(composition)
- 새로은 클래스에 기존 클래스 객체를 참조하는 private 필드를 두자.
- 전달(forwarding) : 새로운 클래스에 포함된 각각의 메서드는 기존 클래스에 있는 메서드 가운데 필요한 것을 호출해서 그 결과를 반환하면 된다. -> 각 메서드를 전달 메서드(forwarding method)라고 한다.

```JAVA
//계승 대신 구성을 사용하는 포장 클래스(wrapper class)
public class InstrumentedSet<E> extends ForwardingSet<E> {
     private int addCount = 0;

     public InstrumentedSet (Set<E> s){
          super(s);
     }

     @Override
     public boolean add(E e){
        addCount++;
        return super.add(e);
     }
}

//재사용 가능한 전달(forwarding) 클래스
public class ForwardingSet<E> implements Set<e>
{
    //구성. 기존 객체를 참조하는 private 필드
     private final Set<E> s;

     public ForwardingSet(Set<E> s){ this.s = s;}

     public boolean add(E e){
       return s.add(e);
     }

     public boolean addAll(Collection<? extends E> c) {
       return s.addAll(c);
     }
}
```
- InstrumentedSet을 이렇게 설계할 수 있는 것은, HashSet이 제공해야 할 기능을 규정하는 Set이라는 인터페이스가 있기 때문!
- 안정적이고 유연성도 높다.
- InstrumentedSet 클래스는 Set 인터페이스를 구현하며, Set 객체를 인자로 받는 생성자를 하나 가지고 있다.
- InstrumentedSet 클래스가 Set 객체를 인자로 받아 필요한 기능을 갖춘 다른 Set 객체로 변환
- 포장 클래스(wrapper class) 기법을 쓰면 어떤 원하는 Set 구현도 원하는 대로 수정할 수 있고, 이미 있는 생성자도 그대로 사용 가능

- 포장 클래스의 단점 : 역호출 프레임워크(call back)와 함게 사용하기는 부적절
 - 포장된 객체는 포장 객체에 대해서 모르기 때문에, 자기 자신에 대한 참조(this)를 전달할 것이다.


### 4. 클래스 B는 클래스 A와 "IS-A" 관계가 성립할 때만 A를 계승해야 한다.
- 계승은 하위 클래스가 상위 클래스의 하위 자료형(subtype)이 확실한 경우에만 바람직하다.
- 원칙 위반 사례 : stack 과 vector (stack은 vector를 계승하면 안된다. 구성 기법이 더 적절하다.)
- 구성 기법이 적절한 곳에 계승을 사용하면 구현 세부사항이 쓸데없이 노출된다.
- 가장 심각한 문제 : 클라이언트가 상위 클래스 상태를 직접 변경하여 하위 클래스의 불변식을 깰 수 있다.

### 요약
- 계승은 강력한 도구이지만 __캡슐화 원칙을 침해__하므로 문제 발생 소지가 있다.
- 상위 클래스와 하위 클래스 사이에 __IS-A 관계__가 있을 때만 사용하자.
- IS-A 관계가 성립해도, 하위 클래스가 상위 클래스와 다른 패키지에 있거나 계승을 고려해 만들어진 상위클래스가 아니면 깨지기 쉽다. __-> 구성과 전달기법 사용__
- 포장 클래스(wrapper class)는 하위 클래스보다 견고할 뿐 아니라, 강력하다.
