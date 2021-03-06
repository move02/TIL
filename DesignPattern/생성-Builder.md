생성 - Builder
=========

> 날짜 : 2020.02.14

## Builder 패턴
다른 생성쪽 패턴과 마찬가지로 객체를 생성하기 위해 사용되는 패턴.
Lucene을 활용하면서 lucene query 객체를 생성하는 패턴이 빌더패턴이라고 할 수 있다.
```java
QueryBuilder queryBuilder = fullTextSession.getSearchFactory() 
    .buildQueryBuilder()
    .forEntity(News.class)
    .get();

org.apache.lucene.search.Query fuzzyQuery = queryBuilder
            .keyword()
            .fuzzy()
            .withEditDistanceUpTo(1)
            .withPrefixLength(0)
            .onField("body")
            .matching(keyword)
            .createQuery();
```

### Builder 패턴을 사용하는 이유
- **핵심 목적 : 객체의 생성 알고리즘과 조립 방법을 분리하는 것이 목적**.
  - Immutable 한 객체를 유연하게 생성하기 위해.
  - 코드 가독성이 높아짐.

### 생성자, setter VS Builder
다음과 같은 Pizza 클래스가 있다고 하자.
```java
public class Pizza {
    private final String menuName; // 필수
    private final String size;     // 선택
    private final double price;    // 선택
}
```

#### 생성자 패턴

생성자를 이용해 객체를 만드는 방법은 다음과 같다.
```java
public class Pizza {
    private final String menuName; // 필수
    private final String size;     // 선택
    private final double price;    // 선택
    
    // 이름만 인자로 받는 생성자
    public Pizza(String menuName){
        this(menuName, "M", 20.00);
    }

    // 이름과  인자로 받는 생성자
    public Pizza(String menuName, String size){
        this(menuName, size, 20.00);
    }

    public Pizza(String menuName, double size, double price){
        this.menuName = menuName;
        this.size = size;
        this.price = price;
    }
}

// 기본 피자 생성
new Pizza("포테이토 피자");
// 크게 만들고 싶을 때.
new Pizza("포테이토 피자", "L");
// 크게 만들어서 비싸게 받을 때
new Pizza("포테이토 피자", "XL", 25.00);
```

##### 장점
- ~~객체를 처음 만들 때 멤버변수를 바로 초기화할 수 있다는 장점이 있다.~~
- 딱히 없는 것 같다.

##### 단점
- 인자가 추가되거나 삭제되면 코드의 수정 또는 삭제가 어렵다.
- 코드의 가독성이 떨어지며, 특히 인자의 수가 많을 때에는 각 부분이 어떤 인자를 의미하는지 알기 어렵다.

##### 결론
가변성이 있는 객체에 사용하기엔 적절하지 않은 방법.

#### Setter
Setter를 이용해 객체를 만드는 방법은 다음과 같다.(JavaBeans Pattern)
```java
Pizza potatoPizza = new Pizza();
potatoPizza.setMenuName("포테이토 피자");
potatoPizza.setSize("XL");
potatoPizza.setPrice(25.00);
```

##### 장점
- 각 인자의 의미를 파악하기 쉽다.
- 멤버 변수의 추가, 삭제에 대응이 용이하다.

##### 단점
- 객체 일관성이 깨질 수 있다 -> 초기에 정한 값이 언제든 바뀔 수 있다.
- `setter` 메소드 때문에 immutable 클래스를 만드는 데에는 적절하지 않다.
  - [Thread Safe](../기타/ThreadSafe.md)를 위해 생성자 패턴보다 많은 일을 해야 한다. (객체 생성 및 초기 할당을 여러 번 해야한다는 뜻.)

#### Builder
Builder 패턴을 이용해 객체를 만드는 방법은 다음과 같다.
```java
public class Pizza {
    private final String menuName; // 필수
    private final double price;    // 선택
    private final String size;     // 선택
    private final String dough;    // 선택

    public static class Builder {
        // 필수 인자들.
        private final String menuName; // 필수

        // 선택적 인자들 -> 기본 값으로 초기화를 해주어야 한다.
        private double price = 20.00;    
        private String size = "M";
        private String dough = "Original";

        public Builder(String menuName){
            this.menuName = menuName;
        }

        public Builder price(double price){
            this.price = price;
            return this;
        }
        public Builder size(String size){
            this.size = size;
            return this;
        }
        public Builder dough(String dough){
            this.dough = dough;
            return this;
        }
        public Pizza build(){
            return new Pizza(this);
        }
    }

    private Pizza(Builder builder){
        this.menuName = builder.menuName;
        this.price = builder.price;
        this.size = builder.size;
        this.dough = builder.dough;
    }
}
```

클래스 구조가 조금 복잡하지만 다음과 같이 사용이 가능하다.
```java
Pizza potatoPizza = new Pizza
                            .Builder("포테이토")
                            .price(25.00)
                            .size("XL")
                            .dough("Napoli")
                            .build();
```

##### 장점
- 각 인자가 어떤 의미인지 알기 쉽다.
- immutable 한 객체를 만들 수 있다.
- 객체 일관성이 깨지지 않는다.
- `build()` 함수를 통해 잘못된 값이 입력되었는지 검증도 가능하다.

인터페이스를 사용한다면 조금 더 유연하게 빌더 패턴의 사용이 가능하다.
```java
public interface Builder<T>{
    public T build();
}

// Pizza의 빌더클래스가 위의 인터페이스를 구현
Builder<Pizza> builder = new Builder<>();

Pizza potatoPizza = builder("포테이토")
                            .price(25.00)
                            .size("XL")
                            .dough("Napoli")
                            .build();
```


### 참고
- <https://johngrib.github.io/wiki/builder-pattern/>