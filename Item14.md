# 아이템 14. Comparable을 구현할지 고려하라


### ❄️ **compareTo가 equals와 가진 차이점**

- 단순 동치성 비교에 더해 순서까지 비교 할 수 있다.
- Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다.
- 타입이 다른 객체를 신경쓰지 않아도 된다. (타입이 다른 객체가 주어지면 `ClassCastException` 던져도 된다)

### ❄️ Comparable을 구현하면 큰 효과를 얻을 수 있다

Comparable을 구현한 객체들은 손쉽게 정렬할 수 있고 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 할 수 있다.

```java
public class WordList {
	public static void main(String[] args) {
		Set<String> s = new TreeSet<>();
		Collections.addAll(s, args);
		System.out.println(s);
	}
}
```

위의 코드는 String이 Comparable을 구현했기에 명렬줄 인수들을 중복을 제거하고 알파벳 순으로 출력한다.

**알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.**

### ❄️ 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트 추가 시

기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가 했다면 compareTo 규약을 지킬 방법이 없다.

Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두자.

**<예시 코드>**

```java
class Parent implements Comparable<Parent> {
    private int a;

    @Override
    public int compareTo(final Parent o) {
        return a - o.a;
    }
}

class Child implements Comparable<Child> {
    
    private Parent parent;
    protected int b;

    @Override
    public int compareTo(final Child o) {
        if (b - o.b == 0) {
            return parent.compareTo(o.parent);
        }
        return b - o.b;
    }
}
출처: https://ttl-blog.tistory.com/1230 [Shin._.Mallang:티스토리]
```

### ❄️ compareTo와 equals가 일치하지 않으면 문제가 생길 수 있다.

물론 일관되지 않아도 여전히 동작은 하지만 **정렬된 컬렉션에 넣을 때 문제가 생길 수 있다.**

정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용하기 때문이다.

compareTo와 equals가 일관되지 않은 대표적 예시가 `BigDecimal`이다.

```java
final BigDecimal bigDecimal1 = new BigDecimal("1.0");
final BigDecimal bigDecimal2 = new BigDecimal("1.00");

final Set<BigDecimal> hashSet = new HashSet<>();
hashSet.add(bigDecimal1);
hashSet.add(bigDecimal2);

System.out.println(hashSet.size());  // 2

final Set<BigDecimal> treeSet = new TreeSet<>();
treeSet.add(bigDecimal1);
treeSet.add(bigDecimal2);

System.out.println(treeSet.size());  // 1
```

TreeSet은 정렬된 컬렉션으로 내부에서 동치성 비교를 할 때 compareTo를 사용한다.

### ❄️ compareTo 작성

> **Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수타입은 컴파일 시에 결정된다. 입력 인수 확인이나 형변환을 할 필요가 없다.**
>

> **null을 인수로 넣으면 NullPointerException을 던져야한다.**
>

> **compareTo는 각 필드를 동치가 아닌 순서를 비교한다.**
>

객체의 여러 필드가 아닌 순서를 기준으로 비교된다. 즉, 두 객체를 비교할 때, 먼저 첫 번째 필드를 비교하고, 그 결과에 따라 두 번째 필드로 이동하여 비교하는 식으로 진행한다.

**예제 코드**

```java
javaCopy code
public class Example implements Comparable<Example> {
    private int field1;
    private String field2;

    // 생성자와 다른 메서드들

    @Override
    public int compareTo(Example other) {
        // 먼저 field1을 비교
        int result = Integer.compare(this.field1, other.field1);

        // 만약 field1이 같다면 field2를 비교
        if (result == 0) {
            result = this.field2.compareTo(other.field2);
        }

        return result;
    }
}

```

> **객체 참조 필드를 비교하려면 compareTo 메소드를 재귀적으로 호출한다.**
>

> **Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator를 대신 사용한다.**
>

[자바 [JAVA] - Comparable 과 Comparator의 이해](https://st-lab.tistory.com/243)

### ❄️ 관계 연산자 < 와 > 는 추천하지 않는다

박싱된 기본 타입 클래스들에 존재하는 정적 메서드 compare를 이용하는 것을 추천한다. 관계 연산자 <와 >는 거추장스럽고 ***오류를 유발할 수 있다.***

```java
class MyObject implements Comparable<MyObject> {
    private int a;
    private double b;

    @Override
    public int compareTo(final MyObject o) {
        if (a == o.a) {
            return Double.compare(b, o.b);
        }
        return Integer.compare(a, o.a);
    }
}
```

### ❄️ 클래스에 핵심 필드가 여려개인 경우

어느 필드를 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드부터 비교한다. 비교 결과가 0이 아니라면 이후 필드는 고려하지 않고 거기서 끝마친다.

```java
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if (result == 0) {
		result = Short.compare(prefix, pn.prefix);
		if (result == 0) {
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
}
```

*자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다. 그리고 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데 활용할 수 있다.*

### ❄️ 비교자 생성 메서드를 활용한 비교자

```java
private static final Comparator<PhoneNumber> COMPARATOR =
				comparingInt((PhoneNumber pn) -> pn.areaCode)
						.thenComparingInt(pn -> pn.prefix)
						.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPATATOR.compare(this, pn);
}
```

### ❄️ 값의 차이를 반환하는 비교자는 사용하지 말자

```java
class MyInt implements Comparable<MyInt> {
    private int a;

    @Override
    public int compareTo(final MyInt o) {
        return a - o.a;
    }
}
```

이 방식은 정수 오버플로를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

