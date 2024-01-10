# 아이템 18. 상속보다는 컴포지션을 사용하라

### ❄️ 메서드 호출과 달리 상속은 캡슐화를 위반한다

상속은 **강한 결합**을 가진다. 이것은 **캡슐화를 위반할 가능성이 높다**. 여기서 캡슐화는 SOLID 5원칙에 의거하면 단일 책임 원칙을 지키지 못했을 확률이 높다.

(**객체는 내부에 캡슐화하고 외부에 공개하지 않으면서 스스로의 상태를 책임져야 하는데, 상속이 가진 강결합으로 인해 자신의 책임이 아닌걸 갖게 되면 캡슐화를 위반할 가능성이 있다)**

예를 들어서 부모 클래스와 서브 클래스가 존재할 때, 부모 클래스가 기능을 추가하면 서브 클래스가 필요하지 않은 기능일지라도 그 책임을 갖게 된다. 내가 하고 싶지 않은 일도 super 때문에 어쩔 수 없이 하게 되는 일이 발생한다. 서브 클래스가 변경될 사항이 아닌 일로 변경되는 경우가 발생할 수 있다.

상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다. 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정돼야만 한다.

```java
import java.util.Collection;
import java.util.HashSet;

public class InstrumentdHashSet<E> extends HashSet<E> {

    private int addCount = 0;

    public InstrumentdHashSet() {}

    public InstrumentdHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

이 클래스는 제대로 동작하지 않는다. 이 클래스의 인스턴스에 addAll 메서드로 원소 3개를 더해보자.

```java
InstrumentdHashSet<String> s = new InstrumentdHashSet<>();

s.addAll(List.*of*("1", "2,","3"));
```

getAddCount 메서드를 호출하여 3을 반환하리라 기대하겠지만, 실제로는 6을 반환한다. HashSet의 addAll 메서드는 각 원소를 add 메서드를 호출해 추가하는데, 이때 불리는 add 하위 클래스에서 재정의한 메서드다. 따라서 addCount에 값이 중복해져 더해져 최종값이 6으로 늘어난 것이다.

### ❄️ 조합(Composition)을 사용하자

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.

기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션이라 한다.