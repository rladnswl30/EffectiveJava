## Rule3. private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라
### ___JDK 1.5 이전, 싱글턴 구현 방법 두가지___
#### 1. 정적 멤버는 final로 선언


```JAVA
//public final 필드를 이용한 싱글턴
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding() { ... }
}
```


- private 생성자는 Elvis.INSTANCE를 초기화 할 때 한 번만 호출된다.


<br>
#### 2. public으로 선언된 정적 팩터리 메서드를 이용


```JAVA
//정적 팩터리를 이용한 싱글턴
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding() { ... }
}
```


- Elvis.getInstance는 항상 같은 객체에 대한 참조를 반환한다.
- API를 변경하지 않고도 싱글턴 패턴을 포기할 수 있다.


<br>
### 단점
- 위 두가지 방법들로 구현한 싱글턴 클래스를 직렬화 가능 클래스로 만드려면 클래스 선언에 implements Serializable을 추가하는 것으로는 부족
- 싱글턴 특성 유지를 위해 모든 필드를 transient로 선언하고, readResolve 메서드를 추가해야 한다.


```JAVA
//싱글턴 상태를 유지하기 위한 readResolve 구현
private Object readResolve() {
	//동일한 Elvis 객체가 반환되도록 하는 동시에, 가짜 Elvis 객체는
	//쓰레기 수집기(garbage collector)가 처리하도록 만든다.
	return INSTACNE;
}
```


<br>
### ___JDK 1.5 이후___


#### 1. 원소가 하나뿐인 enum 자료형 정의


```JAVA
//Enum 싱글턴 - 이렇게 하는 쪽이 더 낫다.
public enum Elvis {
	INSTANCE;

	public void leaveTheBuilding() { ... }
}

```


- public 필드 사용 구현법과 동등하다. 한가지 차이는 좀 더 간결, 직렬화가 자동으로 처리된다는 것.
- __요약 :__ 원소가 하나뿐인 enum 자료형이 싱글턴을 구현하는 가장 좋은 방법
