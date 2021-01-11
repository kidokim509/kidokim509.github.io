---
title:   java off heap
date:    2018-10-26 23:59:00 +0900
categories: [dev, java]
tags: [java, jvm, off-heap, unsafe]
---
## Off heap memory allocation
- sun.misc.Unsafe: JVM internal API로서 생성자 호출없이 Object를 생성한다거나 하는 다양한 기능의 API가 있으나
  - 대표적으로 Non-JVM 영역(off-heap, direct memory)에 메모리를 할당하여 사용할 수 있음
    - JVM 영역 밖의 메모리를 사용 ==> GC Overhead가 없음. 대신 leak이 나지 않게 개발자가 메모리 관리를 잘 해야 함
      - 당연히,  Off-heap 메모리 사용량은 JVM Memory 모니터링에 잡히지 않으며, 프로세스 메모리 사용량을 모니터링 해야함
    - Large and Long lived object를 메모리에 올릴 때 이점이 있음
  - Off-heap 메모리를 사용할 수 있는 방법이 Unsafe만 있는 것은 아님
    - java nio의 ByteBuffer 역시 jvm heap외 영역에 메모리 할당. GC의 영향을 받지 않음
    - ![DirectByteBuffer](https://www.ibm.com/developerworks/library/j-nativememory-linux/bytebuffers.gif)
    - -XX:MaxDirectMemorySize=N
      - [https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)
        - Sets the maximum total size (in bytes) of the New I/O (the java.nio package) direct-buffer allocations. ... By default, the size is set to 0, meaning that the JVM chooses the size for NIO direct-buffer allocations automatically.

### Sample Code
```java
public class TestUnsafe {
    Unsafe unsafe;

    public static Unsafe getUnsafe() {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            return (Unsafe) f.get(null);
        } catch(Exception e) {
            throw new RuntimeException("Can not obtain unsafe object.");
        }
    }
    
    @Before
    public void setUp() {
        unsafe = getUnsafe();
    }
    
    class DirectIntArray {
    
        private final static long INT_SIZE_IN_BYTES = 4;
    
        private final long startIndex;
    
        public DirectIntArray(long size) {
            startIndex = unsafe.allocateMemory(size * INT_SIZE_IN_BYTES);
            unsafe.setMemory(startIndex, size * INT_SIZE_IN_BYTES, (byte) 0);
        }
    
        public void setValue(long index, int value) {
            unsafe.putInt(index(index), value);
        }
    
        public int getValue(long index) {
            return unsafe.getInt(index(index));
        }
    
        private long index(long offset) {
            return startIndex + offset * INT_SIZE_IN_BYTES;
        }
    
        public void destroy() {
            unsafe.freeMemory(startIndex);
        }
    }
    
    @Test
    public void testDirectIntArray() throws Exception {
        long maximum = Integer.MAX_VALUE + 1L;
    
        DirectIntArray directIntArray = new DirectIntArray(maximum);
        directIntArray.setValue(0L, 10);
        directIntArray.setValue(maximum, 20);
        assertEquals(10, directIntArray.getValue(0L));
        assertEquals(20, directIntArray.getValue(maximum));
    
        directIntArray.destroy();
    }

```


## 참고
- [https://dzone.com/articles/understanding-sunmiscunsafe](https://dzone.com/articles/understanding-sunmiscunsafe)
- [http://mishadoff.com/blog/java-magic-part-4-sun-dot-misc-dot-unsafe/](http://mishadoff.com/blog/java-magic-part-4-sun-dot-misc-dot-unsafe/)
- [https://www.slideshare.net/mishadoff/unsafe-java-18168628?qid=58c0e7b7-a646-4a10-a25e-f4f7511b8661&v=&b=&from_search=2](https://www.slideshare.net/mishadoff/unsafe-java-18168628?qid=58c0e7b7-a646-4a10-a25e-f4f7511b8661&v=&b=&from_search=2)
