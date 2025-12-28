
## 배열의 toString
``` java
public String toString() {  
    return getClass().getName() + "@" + Integer.toHexString(hashCode());  
} 
```
- 배열의 객체 주소 반환
- 배열을 문자열로 요소를 확인하고 싶을 때에는 `toString()` 메서드 **오버라이딩 필요**
- `toString()` 메서드를 호출할 때마다 기존 객체의 주소를 반환함

## util 패키지의 toString
**AbstractCollection 클래스**
``` java

public String toString() {  
    Iterator<E> it = iterator();  
    if (! it.hasNext())  
        return "[]";  
  
    StringBuilder sb = new StringBuilder();  
    sb.append('[');  
    for (;;) {  
        E e = it.next();  
        sb.append(e == this ? "(this Collection)" : e);  
        if (! it.hasNext())  
            return sb.append(']').toString();  
        sb.append(',').append(' ');  
    }
}
```

**Arrays 클래스**
``` java
public static String toString(Object[] a) {  
    if (a == null)  
        return "null";  
  
    int iMax = a.length - 1;  
    if (iMax == -1)  
        return "[]";  
  
    StringBuilder b = new StringBuilder();  
    b.append('[');  
    for (int i = 0; ; i++) {  
        b.append(String.valueOf(a[i]));  
        if (i == iMax)  
            return b.append(']').toString();  
        b.append(", ");  
    }
}
```

- 리스트와 배열의 요소를 문자열로 반환
- 리스트와 배열의 요소를 루프로 돌면서 콤마와 중괄호로 연결하여 새로운 문자열 객체 생성
- `toString()` 메서드를 호출할 때마다 새로운 문자열 객체가 생성됨

### 문제점
- 로그 레벨이 달라도 Arrays.toString이 먼저 실행되는 경우엔 필요하지 않는데 새로운 객체를 생성하는 문제가 있음
- 예시 코드:
```java 
log.debug("arr={}", Arrays.toString(arr)); // 로그 레벨이 달라도 메서드 자체는 호출하기 때문에 객체 생생됨
```
- 현업에서는 if문으로 어떤 로그레벨인지 확인하고 로그 출력하도록 객체 생성 문제 방지함
- 예시 코드:
```java
if (log.isDebugEnabled()) { // 레벨 가드
    log.debug("arr={}", Arrays.toString(arr)); // if문에서 로그 레벨 체크 후 메서드 호출하기 때문에 객체 생성 X
}
```

