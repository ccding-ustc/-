# List 接口
[TOC]
常见三种 List 接口的实现包括 ArrayList, Vector, LinkedList 。三者都实现了 AbstractList ， AbstractList 实现了 List 接口，并且扩展自 AbstractCollection 。
## ArrayList 和 Vector 
- ArrayList 和 Vector 都使用了数组实现，封装了对数组的添加、删除、插入新元素等操作。
- 两者几乎使用了相同的算法，唯一的区别就是对多线程的支持。
- ArrayList 没有对任何一个方法做线程同步，所以不是线程安全的；而 Vector 中绝大数方法都做了线程同步，所以是线程安全的。
- ArrayList 和 Vector 的性能相差无几，没有实现线程安全的 ArrayList 稍好一点。

## LinkedList 
LinkedList 使用双向循环链表数据结构，每个表项包含三个部分：元素内容、前驱表项和后驱表项。

### <font color="red">数组实现和链表实现的异同 </font>(以 LinkedList 和 Vector 为例)
#### 增加元素至列表尾端
`void add(E e)`

- ArrayList 底层实现
``` java 
public boolean add(E e){
    ensureCapacity(size+1);
    elementData[size++] = e;
    return true
}
```
首先确保有足够的空间，然后将元素添加到数组末尾，其 `add()` 方法的性能取决于 `ensureCapacity()` 方法。
``` java
public void ensureCapacity(int minCapacity){
    modCount++;
    int oldCapacity = elementData.length;
    if(minCapacity > oldCapacity){
        Object oldData[] = elementData;
        int newCapacity = (oldCapacity * 3)/2 + 1;
        if(newCapacity < minCapacity)
            newCapacity = minCapacity;
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```
从上面的实现中可以看出，只有当 ArrayList 对容量的要求超过当前数组的大小时，才需要扩容（扩充到原始容量的1.5倍）。扩容过程中，会进行大量的数组复制。

- LinkedList 底层实现
```
public boolean add(E e){
    addBefore(e, header);
    return true;
}
```
`header` 元素的前驱是列表的最后一个元素，将元素 `e` 插入到 `header` 前，就相当于将元素插入到列表尾部。`addBefore()` 方法实现如下：
```java 
public Entry<E> addBefore(E e, Entry<E> entry){
    Entry<E> newEntry = new Entry<E>(e, entry, entry.previous);
    newEntry.previous.next = newEntry;
    newEntry.next.previous = newEntry;
    size++;
    modCount++;
    return newEntry;
}
```
由于 LinkedList 采用双向循环链表结构，所以不需要维护容量的大小。但是每次元素的增加都需要新建一个 `Entry` 对象，并进行更多的赋值操作，在频繁的系统调用，对性能有一定的影响。<u>由于需要新建对象，所以 LinkedList 对堆内存和 GC 的要求更高。</u>

#### 增加元素到列表任意位置
`void add(int index, E e)`

- ArrayList 底层实现
```java
public void add(int index, E e){
    if(index > size || index < 0)
        throw new IndexOutOfBoundsException(
        "Index: " + index + ", size: " + size);
    ensureCapacity(size+1);
    System.arraycopy(elementData, index, elementData, index+1, size-index);
    elementData[index] = e;
    size++;
}
```
从中可以看出，每次插入操作都会进行一次数组复制。大量的数组复制操作会导致系统性能下降。并且，插入的位置越靠前，数组复制的开销越大。

- LinkedList 底层实现
```java
public void add(int index, E e){
    addBefore(element, (index==size ? header : entry(index)));
}
```
<del>对于 LinkedList 来说，插入任意位置和插入尾部是一样的，不会因为插入的位置而影响性能。</del>书中此描述存在问题，同删除任意位置元素一样，当插入位置位于中间时， LinkedList需要遍历半个 List 找到插入的位置，此时，比插入在前端或者末端的性能要差很多。

#### 删除任意位置元素
`public E remove(int index)`

- ArrayList 底层实现
对于 ArrayList 来说，删除任意位置的元素和插入任意位置的元素没有本质区别。在任意位置删除元素，需要对数组进行复制操作。
```java
public E remove(int index){
    RangeCheck(index);
    modCount++;
    E oldValue = (E) elementData[index];
    int numMoved = size - index - 1;
    if(numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null;
    return oldValue;
}
```
同样，删除的元素越靠前，系统开销越大。

- LinkedList 底层实现
```java
public E remove(int index){
    return remove(entry(index));
}

private Entry<E> entry(int index){
    if(index < 0 || index >= size)
        throw new IndexOutOfBoundsException("Index: " + index + ", size: " + size);

    Entry e = header;
    if(index < (size >> 1){
        for(int i=0; i<=index; i++){
            e = e.next;
        }
    }else{
        for(int i=size; i>index; i--){
            e = e.presvious;
        }
    }
    return e;
}
```
删除任意位置元素，首先需要通过循环找到删除的位置。如果位置在 List 的前半段，则从前开始找，反之，则从后开始找。因此，无论要删除较前或者较后位置的元素都比较高效；但是如果删除中间的元素，则需要遍历几乎半个 List ，在 List 拥有大量元素时，效率较低。
#### 容量参数
容量参数是 ArrayList 和 Vector 等基于数组实现的 List 的特有性能参数，它表示初始化数组的大小。由上面可知，当 ArrayList 所存储的元素数量超过其已有大小时，它便会进行扩容，数组的扩容会导致整个数组进行一次复制。合理的数组大小会减少数组扩容的次数，从而提高系统的性能。
默认情况下， ArrayList 的初始数组大小是 10 ，每次扩容将新的数组大小设置为原数组大小的 1.5 倍。
```java
public ArrayList(){
    this(10);
}
public ArrayList(int initialCapacity){
    super();
    if(initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    this.elementData = new Object[initialCapacity];
}
```
#### 遍历列表
JDK 1.5 以后，列表有三种遍历方式：ForEach 操作、迭代器和 for 循环。综合性能迭代器略好于 ForEach 操作。使用 for 循环通过随机访问遍历列表，所以基于数组实现的 List 性能表现都很好，但是对于 LinkedList 性能表现很差，因为对 LinkedList 进行随机访问时，总会进行一次列表的遍历操作。
