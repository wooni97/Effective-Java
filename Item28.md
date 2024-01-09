# 아이템 28. 배열보다는 리스트를 사용하라

### ☑️ 배열은 공변, 제네릭은 불공변이다

**배열은 공변이다.** Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.

**제네릭은 불공변이다.** 서로 다른 타입 Type1, Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

Type1이 Type2의 하위 타입일지라도 List<Type1>, List<Type2> 어떠한 관계도 갖지 않는다.

### ☑️ 배열은 런타임에서 타입 안전성을 갖지 못한다

어떤 배열 형태일때 런타임에 타입이 안정적이지 못할까? 바로 **`Object 배열`**이다.

- 런타임에 실패하는 코드

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; //ArrayStoreException을 던진다
```

- 컴파일되지 않는 코드

```java
List<Object> ol = new ArrayList<Long>(); //호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```

배열에서 타입 실수를 하게 되면 그 실수를 런타임에서야 알게 되지만, 리스트를 사용하면 컴파일 타임에 바로 알 수 있다.

### ☑️  배열은 실체화된다

배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.

```java
Object[] objects = new String[1];
```

배열은 런타임에 타입이 실체화되기 때문에 objects는 런타임에 String[]가 된다.

**그러나 제네릭은 타입 정보가 런타임에는 소멸된다.** 원소 타입을 컴파일타임에만 검사하며 런타임에는 알수조차 없다.

```java
// 컴파일 타임(실제 작성한 코드)
ArrayList<String> stringList = new ArrayList<String>();
ArrayList<Integer> integerList = new ArrayList<Integer>();

// 런타임(제네릭 타입은 런타임에 소거되므로 구분이 불가능하다)
ArrayList stringList = new ArrayList();
ArrayList integerList = new ArrayList();
```

그렇기에 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.

즉, 코드 `new List<E>[]` , `new List<String>[]` , `new E[]` 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다.

> **제네릭 배열을 만들지 못하게 막은 이유는?**
>
- **타입 안전하지 않기 때문이다**. 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 `ClassCastExecption`이 발생할 수 있다.

```java
//제네릭 배열을 생성하는 코드가 허용된다고 가정해보자

List<String>[] stringLists = new List<String>[1];  //(1) 제네릭 배열을 생성. 런타임시에는 제네릭 타입은 소거되므로 ArrayList[]가 된다.
List<Integer> intList = List.of(42);               //(2) 타입 소거로 인해 런타임시 ArrayList가 된다.
Object[] objects = stringLists;                    //(3) 배열은 공변성을 가지므로 Object[]는 ArrayList[]가 될 수 있다.
objects[0] = intList;                              //(4) intList 또한 ArrayList이므로 배열의 요소가 될 수 있다.
String s = strigLists[0].get(0);                   //(5) String 타입을 가져야 하지만 Integer이므로 예외가 발생한다.
```

### ☑️  제네릭 배열 사용하기

> **와일드카드 타입을 이용한 제네릭 배열 생성**
>

```java
// 컴파일 에러가 발생 안함.
List<?>[] lists = new List<?>[2];
lists[0] = Arrays.asList(1);
lists[1] = Arrays.asList("A");
    for (List<?> list : lists) {
        System.out.println(list);
    }
}
```

- 비한정적 와일드카드 타입으로 제네릭 배열을 생성하면 컴파일 에러가 발생하지 않는다.
- 비한정적 와일드카드 타입은 모든 제네릭 타입을 가질 수 있다. 그러므로 raw 타입으로 정의한 `List[]` 와 동일한 의미를 가지므로 컴파일 에러를 발생시키지 않는다.

> **형변환 이용하기**
>

```java
public class Store<E> {
    private E[] elements;
    private int index;
    
    // 경고가 발생하나 타입 안전성을 확신할 수 있으니 경고를 제거한다.
    @SuppressWarnings("unchecked")
    public Store(int size) {
     // this.elements = new E[size]; 직접 제네릭 배열은 생성불가!
        this.elements = (E[]) new Object[size]; // 강제 형변환을 이용하여 생성
        this.index = 0;
    }
    
    // elements는 해당 메서드에서만 추가될 수 있으므로 코드상으로 타입 안전성 보장이 가능하다. 
    public boolean save(E e) {
        if (index >= elements.length)
            return false;
        
        elements[index++] = e;
        return true;
    }
}
```

- Object 배열에서 (E[])로 형변환을 이용하면 제네릭 배열을 만들 수 있다.
- 형변환을 통해 제네릭 배열을 생성하는 경우 타입 안전성을 보장해줄 수 없기 때문에 컴파일러에서 경고를 표시한다.
- 하지만 코드 상으로 타입 안전성 보장이 가능하다면 문제 없이 사용할 수 있다.