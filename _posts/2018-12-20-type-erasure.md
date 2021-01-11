---
title: type erasure
date:  2018-12-20 02:59:00 +0900
categories: [dev, java]
tags: [java, generic]
---
참고: 
* [https://www.baeldung.com/java-type-erasure](https://www.baeldung.com/java-type-erasure)
* [http://toby.epril.com/?p=248](http://toby.epril.com/?p=248)
* [http://wonwoo.ml/index.php/post/1743](http://wonwoo.ml/index.php/post/1743)

Type erasure can be explained as the process of enforcing type constraints only at compile time and discarding the element type information at runtime.
타입 제한을 컴파일 타임에만 강제하고, 런타임에는 타입 정보를 제거해버린다.

```java
public static  <E> boolean containsElement(E [] elements, E element){
    for (E e : elements){
        if(e.equals(element)){
            return true;
        }
    }
    return false;
}
```
이 코드가 컴파일이 되면
이렇게, unbounded type E를 실제 타입인 Object로 바꾼다. 
```java
public static  boolean containsElement(Object [] elements, Object element){
    for (Object e : elements){
        if(e.equals(element)){
            return true;
        }
    }
    return false;
}

```

type parameter가 bounded인 경우라면
```java
public class BoundStack<E extends Comparable<E>> {
    private E[] stackContent;
 
    public BoundStack(int capacity) {
        this.stackContent = (E[]) new Object[capacity];
    }
 
    public void push(E data) {
        // ..
    }
 
    public E pop() {
        // ..
    }
}
```

first bound class로 치환한다.
```java
public class BoundStack {
    private Comparable [] stackContent;
 
    public BoundStack(int capacity) {
        this.stackContent = (Comparable[]) new Object[capacity];
    }
 
    public void push(Comparable data) {
        // ..
    }
 
    public Comparable pop() {
        // ..
    }
}
```

왜? Java Generic은 JVM 레벨의 호환성을 위해서 컴파일시에 Type 정보를 삭제해버린다.
정말 이 이유가 다일까?
