# 아이템 11.  equals를 재정의하려거든 hashCode도 재정의하라

---

### ❄️ **equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.**

그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 hashMap이나 hashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

### ❄️ **논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

> equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
>

위는 Object 명세에서 발췌한 규악이다. hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항이다. (*equals가 두 객체를 다르다고 판단했을땐 hashCode가 같든 다르든 문제가 없다.)*

**논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

**예시)**

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```

이 코드 다음에

```java
m.get(new PhoneNumber(707, 867, 5309));
```

이 코드를 실행하면 “제니”가 나와야 할 것 같지만, 실제로는 null을 반환한다.

PhoneNumber 클래스는 hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다.

이 문제는 hashCode 메서드만 적절히 작성해주면 해결된다.

❌ **최악의(하지만 적법한) hashCode 구현 - 사용 금지**

```java
@Override
public int hashCode() { return 42; }
```

이 코드는 동치인 모든 객체에서 똑같은 해시코들르 반환하니 적법하다. 하지만 끔찍하게도 모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트처럼 동작한다.

그 결과 평균 수행시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 객체가 많아지면 도저히 쓸 수 없게 된다.

**좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다.**

*이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.*

### ❄️ **좋은 hashCode를 작성하는 요령**

### ❄️ 해시코드 캐싱

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다.

```java
public final class ImmutableClass {
    private final int field1;
    private final String field2;
    private int cachedHashCode; // 캐싱된 해시코드

    public ImmutableClass(int field1, String field2) {
        this.field1 = field1;
        this.field2 = field2;
        this.cachedHashCode = computeHashCode();
    }

    private int computeHashCode() {
        // 여기서 해시코드를 계산하는 비용이 큰 작업 수행
        // ...

        return result; // 계산된 해시코드 반환
    }

    @Override
    public int hashCode() {
        return cachedHashCode; // 캐싱된 해시코드 반환
    }

    // equals 메서드 등도 불변 클래스에 맞게 구현해야 함
    // ...
}
```

이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.