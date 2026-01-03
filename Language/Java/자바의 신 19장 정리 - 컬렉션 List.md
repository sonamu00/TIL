## 컬렉션이란
- 목록성 데이터를 처리하는 자료구조
- `Collection` 인터페이스는 `Interable<E>` 인터페이스를 확장함
	- 컬렉션에서 `Interable` 인터페이스를 사용하여 데이터를 순차적으로 가져올 수 있음

### 컬렉션 for-each 의 문제점
- for-each 문은 컬렉션의 Iterator를 사용하여 요소를 순회하도록 함
	=> 컴파일 단계에서 for-each를 `Iterator iter = list.iterator(); while(iter.hasNext())` 으로 변경
- Iterator 는 컬렉션을 순회하는 도중 컬렉션이 수정되면 `ConcurrentModificationException` 예외를 던짐
	=> 지금 내가 보고 있는 컬렉션 상태가 그대로 유지된다는 전제(= 스냅샷)를 갖고 순회함

#### ConcurrentModificationException 예외가 발생하는 흐름
1. for-each 문 안에서 컬렉션 수정
```java
for (String s : list) {
    if (s.startsWith("x")) {
	    list.remove(s); // ConcurrentModificationException 발생
    }
}
```
2. 컴파일 단계에서 `for (String s : list)` 구문을 `Iterator iter = list.iterator(); while(iter.hasNext())` 으로 변경
3. 런타임에서 컬렉션 루프를 돌기 위해 iterator 객체 생성
``` java
public class ArrayList<E> extends AbstractList<E>implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

	public Iterator<E> iterator() { // for-each 구문에서 이 메서드 호출함
	    return new Itr();  
	}
}
```
4. 컬렉션 내부 클래스로 `Iterator` 를 구현한 `next()` 메서드에서 컬렉션이 변경 되었는지 확인
```java
public class ArrayList<E> extends AbstractList<E>implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
	private class Itr implements Iterator<E> {  
	    int cursor;       // index of next element to return  
	    int lastRet = -1; // index of last element returned; -1 if no such  
	    int expectedModCount = modCount;  
	  
	    // prevent creating a synthetic constructor  
	    Itr() {}  
	  
	    public boolean hasNext() {  
	        return cursor != size;  
	    }  
	    @SuppressWarnings("unchecked")  
	    public E next() {  
	        checkForComodification();  
	        int i = cursor;  
	        if (i >= size)  
	            throw new NoSuchElementException();  
	        Object[] elementData = ArrayList.this.elementData;  
	        if (i >= elementData.length)  
	            throw new ConcurrentModificationException();  
	        cursor = i + 1;  
	        return (E) elementData[lastRet = i];  
	    }
	}
}

```
- `modCount`: 리스트가 구조적으로 몇 번 바뀌었는지 카운트함 (AbstractList 계열이 들고 있음)
- `expectedModCount`: Iterator가 생성되는 시점의 컬렉션 수정 횟수 값을 저장, Iterator 사용 도중에 수정이 일어났는지 체크하기 위해 존재
5. `checkForComodification()` 메서드에서 `modCount` 값과 `expectedModCount` 값이 같지 않으면 `ConcurrentModificationException` 예외를 던짐
``` java
final void checkForComodification() {  
    if (modCount != expectedModCount)  
        throw new ConcurrentModificationException();  
}
```


## List 인터페이스란
- `Collection` 인터페이스를 확장함
- 배열처럼 순서가 존재
- 주로 ArrayList, Vector, Stack, LinkedList 구현체들을 사용함

## ArrayList 란
- 확장 가능한 배열
- 생성자 종류
	- `ArrayList()`: 객체를 저장할 공간 10개인 ArrayList를 만듬
		- 저장할 공간을 지정하지 않으면 기본으로 10개 생성
	- `ArrayList(Collection<? extends E> c)`: 매개변수 객체를 `Object` 클래스의 `toArray()` 메서드로 Object 배열로 만든 후, Object 배열 크기만큼 저장 공간을 갖는 ArrayList를 생성
		- 제네릭을 사용하여 컴파일 시점에 타입을 잘못 지정한 부분이 걸러짐
	- `ArrayList(int initialCapacity)`: 매개변수로 넘어온 initialCapacity 값만큼 저장 공간을 갖는 ArrayList를 생성
		- ArrayList 의 초기 크기를 지정
- `Length()` 와 `size()` 의 차이점
	- `Length()`: 배열의 길이, 비어있는 원소(null)도 출력
	-  `size()`: 현재 들어있는 원소 개수 출력

### 초기 크기를 지정하는 이유
- 대량으로 리스트에 값을 추가할 때, 내부 배열에 넣을 공간이 없으면 더 큰 배열을 만들어서 기존 배열을 새 배열로 복사함 => 확장 순간에 원소 개수에 따라 `O(n)` 만큼 비용이 튐
- ArrayList 내부 소스:
```java
boolean add(E e) {
    if (size == elementData.length) {   // capacity 꽉 참
        grow();                         // 1) 더 큰 새 배열 만들고
    }
    elementData[size] = e;              // 4) 그 다음 새 요소 저장
    size++;
    return true;
}

void grow() {
    int newCap = newCapacity(oldCap);           // 보통 1.5배 정도(정확한 정책은 스펙에 고정 아님)
    Object[] newArr = new Object[newCap];       // 1) 새 배열 할당
    System.arraycopy(elementData, 0, newArr, 0, size); // 2) 기존 요소 복사
    elementData = newArr;                       // 3) 참조 교체
}
```
- 참조 교체 후엔 기존 배열은 GC 대상이 됨
- 초기 크기를 크게 잡으면 null 참조 칸이 많아져서 참조 크기만큼 메모리를 할당 받음
	- 장점: 리사이즈(재할당+복사) 횟수 ↓
	- 단점: 안 쓰는 참조 슬롯이 많아져서 메모리 사용량 ↑
	=> 적절히 초기 크기를 할당하는 것이 중요
