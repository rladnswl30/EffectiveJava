# Rule2. 생성자 인자가 많을 때는 Builder 패턴 적용을 고려하라
## 1. 점층적 생성자 패턴
 - 필수 인자만 받는 생성자를 하나 정의하고, 선택적 인자를 하나 받는 생성자를 추가하고,  
   거기에 두개의 선택적 인자를 받는 생성자를 추가하는 식으로, 생성자를 쌓아 올리듯 추가하는 것이다.
	<br>

	```JAVA
	//점층적 생성자 패턴 - 더 많은 인자 개수에 잘 적응하지 못한다
	public class NutritionFacts {
		private final int servingSize;	//(mL)
		private final int servings;		//(per container)
		private final int calories;		//(kcal)
		private final int fat;			//(g)

		public NutritionFacts(int servingSize, int servings) {
			this(servingSize, servings, 0);
		}

		public NutritionFacts(int servingSize, int servings, int calories) {
			this(servingSize, servings, calories, 0);
		}

		public NutritionFacts(int servingSize, int servings, int calories, int fat) {
			this(servingSize, servings, calories, fat, 0);
		}
	}
	```
	<br>
   
	이 클래스로 객체를 생성할 때는 설정하려는 인자 개수에 맞는 생성자를 골라 호출해준다.  

	```JAVA
	NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 3, 35, 27);
	```
   <br>
 
 - __요약 :__ 점층적 생성자 패턴은 잘 동작하지만 인자 수가 늘어나면 클라이언트 코드를 작성하기 어려워지고, 무엇보다 읽기 어려운 코드가 되고 만다.
 <br>

 ## 2. 자바빈 패턴
  - 인자 없는 생성자를 호출하여 객체부터 만든 다음, 설정 메서드들을 호출하여 필수 필드 뿐 아니라 선택적 필드의 값들까지 채우는 것이다.
    <br>

	```JAVA
	//자바빈 패턴 - 일관성 훼손이 가능하고, 항상 변경 가능하다.
	public class NutritionFacts {
		//필드는 기본값으로 초기화(기본 값이 있는 경우만)
		private int servingSize = -1;	//필수 : 기본값 없음
		private int servings = -1;		//필수 : 기본값 없음
		private int calories = 0;
		private int fat = 0;

		public NutritionFacts() { }
		//설정자(setter)
		public void setServingSize(int val) { servingSize = val; }
		public void setServings(int val) { servings = val; }
		public void setCalories(int val) { calories = val; }
	}
	```
	<br>

	작성해야하는 코드의 양이 조금 많아질 수는 있지만 객체를 생성하기 쉬우며, 읽기 좋다.  

	```JAVA
	NutritionFacts cocaCola = new NutritionFacts();
	cocaCola.setServingSize(240);
	cocaCola.setServings(8);
	cocaCola.setCalories(100);
	```
   <br>

 - 1회의 함수 호출로 객체 생성을 끝낼 수 없으므로, 객체 일관성이 일시적을 깨질 수 있다는 단점이 있다.
 - 자바빈 패턴으로는 변경 불가능 클래스를 만들 수 없다.
<br>

## 3. 빌더 패턴
 - 점층정 생성자 패턴의 안전성 + 자바빈 패턴의 가독성
 - 필요한 객체를 직접 생성하는 대신, 클라이언트는 먼저 필수 인자들을 생성자에(or 정적 팩터리 메서드에) 전부 전달하여 빌더 객체를 만든다.
 - 그 후, 빌더 객체에 정의된 설정 메서드들을 호출하여 선택적 인자들을 추가해 나간다.
 - 그리고 마지막으로 아무런 인자 없이 build 메서드를 호출하여 변경 불가능 객체를 만든다.

	<br>

	```JAVA
	public class NutritionFacts {
		private final int servingSize;
		private final int servings;
		private final int calories;
		private final int fat;

		public static class Builder {
			//필수인자
			private final int servingSize;
			private final int servings;

			//선택적 인자 - 기본값으로 초기화
			private int calories = 0;
			private int fat = 0;

			public Builder(int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings = servings;
			}

			public Builder calories(int val) {
				calories = val;
				return this;
			}

			public Builder fat(int val) {
				fat = val;
				return this;
			}
		}

		private NutritionFacts(Builder builder) {
			servingSize = builder.servingSize;
			servings = builder.servings;
			calories = builder.calories;
			fat = builder.fat;
		}
	}

	```
	<br>
	NutritionFacts 객체가 변경 불가능하다는 사실, 모든 인자의 기보낙ㅄ이 한곳에 모여있다는 것에 유의!  

	```JAVA
	NutritionFacts cocaCola = new NutritionFacts.builder(240, 8).calories(100).build();
	```
   <br>

 - Ade나 Python같은 언어는 선택적 인자에 이름을 붙일 수 있도록 허용하는데, 이것과 비슷하다.
 - 생성자와 마찬가지로, 인자에 불변식을 적용할 수 있다.
 - 빌더 객체는 여러개의 varargs 인자를 받을 수 있다.
 - 빌더 패턴은 유연하다.
 - 훌륭한 추상적 팩터리다.
 - __요약 :__ 빌드패턴은 인자가 많은 생성자나 정적 팩터리가 필요한 클래스를 설계할 때, 특히 대부분의 인자가 선택적 인자인 상황에 유용하다.
