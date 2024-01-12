# 아이템 42. 익명 클래스보다는 람다를 사용하라

[익명 클래스와 람다(Lambda)](https://www.notion.so/Lambda-cf954c5aa87a48c193075cfbc6bfd941?pvs=21)

예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용했다. 이런 인터페이스의 인스턴스를 함수 객체라고 하여, 특정 함수나 동작을 나타내는 데 썼다.

JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단은 익명 클래스가 되었다.

### ☘️ 익명 클래스의 인스턴스를 함수 객체로 사용하는것은 낡은 기법이다

```java
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
}
```

익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.

자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다. 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있게 된 것이다.

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

### ☘️ 익명 클래스를 대체하여 람다식을 함수 객체로 사용

```java
Collections.sort(words,
					(s1, se) -> Integer.compare(s1.length(), s2.length()));
```

여기서 람다, 매개변수(s1, s2), 반환값의 타입은 각각(Comparator<String>), String, int 지만 코드에서는 언급이 없다. **컴파일러가 대신 타입을 추론**해준 것이다.

**타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 배개변수 타입은 생략하자.**

### ☘️ 람다 활용하기

람다를 언어 차원에서 지원하면서 기존에는 적합하지 않았던 곳에서도 함수 객체를 실용적으로 사용할 수 있게 되었다.

> **enum Operation**
>
- 람다 활용 전

```java
enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract  double apply(double x, double y);
}

```

람다를 이용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.

- 람다 활용 후

```java
enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

### ☘️ 람다를 사용하지 말아야 하는 경우

**메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.**

람다는 한 줄 일 때 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다. 세 줄을 넘어가면 가독성이 심하게 나빠진다. 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링해야 한다.