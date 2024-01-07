# 아이템 8. finalizer와 cleaner 사용을 피하라

자바가 제공하는 두 가지 객체 소멸자

- finalize
- cleaner

### **finalizer**

Object 클래스는 finalize 메서드를 제공한다. finalize 메소드는 자바에서 객체가 더 이상 참조되지 않을 때 Garbage Collector가 불필요한 메모리를 회수 하면서 실행하는 메소드이다.

**finalize 메소드는 JAVA 9부터 Deprecated되었으며 사용하는 것을 지양하자**

```java
@Deprecated(since="9")
protected void finalize() throws Throwable { }
```

### **cleaner**

finalize를 쓰는게 느리다. 그래서 GC를 할 때 객체 소멸할때 소요되는 시간이 있다. 그래서 대안으로 나온것이 cleaner.

직접 인스턴스도 생성해야하고, clean 메소드도 호출 해야 한다.

GC가 일어날 경우, 등록한 스레드에서 정의된 클린 작업을 수행한다.

### 피해야 하는 이유

> **예측 불가능성 문제**
>

finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다. finalizer, cleaner의 수행은 전적으로 GC 알고리즘에 달렸으며, 이는 천차만별이다.

finalizer 스레드는 다른 애플리케이션 스레드보다 우선순위가 낮아서 실행될 기회를 제대로 얻지 못할 수 있다. → **큐에 쌓여 OutOfMemory 발생 가능성**

cleaner는 자신을 수행할 스레드를 제어할 수 있다는 면에서 조금 낫지만 여전히 백그라운드에서 수행되며 GC의 통제하에 있으니 즉각 수행되리라는 보장은 없다.

`System.gc`를 사용할 수도 있지만 실제 시스템에서는 다음과 같은 이유로 명시적으로 호출 해서는 안된다.

1. 비싸다
2. GC를 즉시 트리거 하지 않는다. JVM이 GC를 시작하도록 하는 힌트일 뿐이다.
3. System.runFinalizersOnExit, Runtime.runFilnalizersOnExit

+) finalizer 동작중 발생한 예외는 무시되며 처리할 작업이 남았더라도 그 순간 종료된다.

unchecked Exception이 발생한다면, finalize에 구현한 로직들은 무시된다.

> **성능 문제**
>

try-with-resources로 자신을 닫도록 하는것보다, finalizer와 cleaner을 사용한 객체 파괴가 훨씬 비효율적이다.

> **보안문제**
>

`finalizer` 를 사용한 클래스는 `finalizer`  공격에 노출되어 심각한 보안문제를 일으킬 수 있다.

<aside>
💡 **finalizer 공격**

생성자나 직렬화 과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적인 하위 클래스의 `finalizer` 가 수행될 수 있게 된다.
예를 들어 finalizer가 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다.

</aside>

악의적인 프로그래머가 해당 클래스르 상속하여 finalizer를 오버라이드하고, 그 안에서 악의적인 작업을 수행할 수 있다.

예를 들어 finalizer에서 정적 필드에 자신의 참조를 할당하면, 해당 객체는 계속해서 살아있게 되어 가비지 컬렉터가 이를 수집하지 못하게 된다.

이는 자원 누수나 메모리 누수와 관련되어 문제를 일으킬 수 있다.

```java
public class FinalizerAttack {

    // 정적 필드에 자신의 참조를 할당
    private static FinalizerAttack instance;

    public FinalizerAttack() {
        // 객체 생성 중에 예외 발생
        throw new RuntimeException("Exception in constructor");
    }

    @Override
    protected void finalize() throws Throwable {
        try {
            // 정적 필드에 자신의 참조를 할당
            instance = this;

            // 여기서 다른 악의적인 동작 수행 가능
        } finally {
            super.finalize();
        }
    }

    public static void main(String[] args) {
        try {
            // 객체를 생성하면서 예외 발생
            new FinalizerAttack();
        } catch (Exception e) {
            // 예외를 처리
        }

        // 가비지 컬렉터 호출을 유도
        System.gc();
        System.runFinalization();

        // 객체가 살아있음을 확인
        if (instance != null) {
            System.out.println("Object is still alive!");
        } else {
            System.out.println("Object is collected by the garbage collector.");
        }
    }
}
```

final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자.

### AutoCloseable

파일이나 스레드 등 종료해야 할 자원을 담고 있는 개체의 클래스에서는 `AutoCloseable`을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 `close` 메서드를 호출하면 된다.

`try-with-resources`를 사용하면 자동으로 안전하게 close 메서드가 호출된다.

### finalizer, cleaner의 쓰임새

그럼 대체 finalizer와 cleaner은 어디에 쓰이는 물건일까?

> 자원의 소유자가 close 메소드를 호출하지 않는 것에 대비한 안전망 역할
>

즉시 호출되리란 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 하지 않는 것보다는 낫다.

자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer를 제공한다.

> 네이티브 피어와 연결된 객체
>

네이티브 피어란 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.

네이티브 객체는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못한다.