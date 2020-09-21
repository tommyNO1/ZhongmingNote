## Collection集合接口常用方法

add(Object obj),addAll(Collection coll),size(),isEmpty,clear())

1. contains(Object o): 判断当前集合是否包含obj ，我们在判断时候会调用obj对象所在类的equals()

2. containsAll(Collection coll1): 判断信长coll1中所有元素是否都存在与当前集合中

3. remove(Object o):remove(Collection coll):差集：从当前集合中移除coll中所有元素

4. retainAll(Collection coll):交集：获取当前集合和coll1集合的交集

5. equals(Object o): 对比两个集合是否相等，调用集合中每一个类的equals()

6. hashCode(): 返回当前对象的哈希值toArray(): 将当前集合返回成一个数组，数组类型为object 

   拓展：Arrays.asList()可以将数组转换为集合

7. iterator(): 返回Iterator接口的实例，用于遍历集合元素

   注意：向Collection接口的实现类的对象中添加数据obj时，要求obj所在类要重写equals()

#### 集合元素的遍历，使用迭代器Iterator接口

```java
Iterator iterator =  coll.iterator();
while(iterator.hasNext()){
    System.out.println(iterator.next());
}
```

- 集合每次调用iterator()方法都会生成一个新的迭代器对象，指针永远在第一个集合元素之前
- 内部定义了remove()，可以在遍历的时候，删除集合中的元素，此方法不同于集合直接调用remove()


## Collection接口的子接口List接口（动态数组）

List集合类中元素有序，且可重复

实现类：

1. ArrayList（主要实现类，线程不安全，效率高，底层使用object[]存储）
2. LinkedList（对于频繁插入、删除操作，使用此类效率比ArrayList高，因为底层使用双向链表存储）
3. vector（古老实现类，线程安全，效率低，底层使用object[]存储）

面试题：三者的异同

#### ArrayList源码分析

```java
LinkedList list = new LinkedList();
```

- 底层创建了长度是10的Object[]数组

- 如果此次添加导致底层elementData数组容量不够，则扩容。默认情况下，扩容为原来容量的1.5倍，同时需要将原有数组中的数据复制到新的数组中

- 在jdk1.8中，创建ArrayList的时候，Object[]数组中初始化为{}，在第一次调用add()时，底层才创建了长度为10的数组，后续的添加和扩容操作与jdk7无异

#### LinkedList源码分析

底层创建了一个双向链表，内部声明了Node类型的first和last属性分别储存俩表的头和尾

#### List常用方法

1. void add(int index,Object obj); 在Index位置插入obj元素
2. boolean addAll(int index,Collection coll);从index位置开始将coll中元素加入list中
3. Object get(int index);
4. int IndexOf(Object obj); 返回obj在集合中首次出现的位置，如果不存在返回-1
5. int lastIndexOf(Object obj); 返回最后一次obj在集合中出现的位置
6. Object remove(int index); 移除指定index位置的元素，并返回此元素
7. Object set(int index, Object obj); 设置指定index位置的元素
8. List subList(int fromIndex,int toIndex);返回从fromIndex到toIndex位置左闭右开的子集合
总结：常用方法
增：add()
删：remove()
改：set()
查：get()
插：add(int index,object ele)
长度：size()
遍历：迭代器，增强for循环，普通循环

## Set接口：存储无序的，不可重复的数据

实现类
1. HashSet：作为Set接口的主要实现类；线程不安全；可以存储null值

2. LinkedHashSet：作为HashSet子类，遍历其内部数据时候可以按照添加的顺序遍历

3. TreeSet：使用红黑树作为存贮结构，可以按照添加对象的指定属性进行排序

#### Set接口中没有二外定义新的方法，使用的都是Collection中声明过的方法
- 无序性：不等于随机性。存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的。
- 不可重复性：保证添加的元素按照equals()判断时，不能返回true，即相同元素只能添加一个

#### 添加元素过程：以HashSet为例
我们向HasSet中添加元素a，首先调用元素a所在类的hashCode()方法，计算元素a的哈希值，此哈希值接着通过某种算法计算出在HashSet底层数组中的存放位置（即为：索引位置），判断
- 如果此位置上没有其他元素，则a添加成功
- 如果此位置上有其他元素b（或以链表形式存在的多个元素），则比较元素a与元素b的hash值：
- 如果hash值不相同，则元素a添加成功
- 如果hash值相同，进而需要调用元素a所在类的equals()方法：
- equals()返回true，元素a添加失败
- equals()返回false，元素a添加成功

对于添加成功的情况2和情况3而言：元素a与已存在指定索引位置上数据以链表的方式存储。
- jdk7：元素a放在数组中，指向原来的元素
- jdk8：原来的元素在数组中，指向元素a
- 总结：七上八下

要求：对于存放在Set容器中的对象，对应的类一定要重写equals()和hashCode(Object obj)方法，以实现对象相等规则，即相等对象必须具有相等的散列码

#### LinkedHashSet
LinkedHashSet作为HashSet的子类，在添加数据的同时，每个数据还维护了两个引用，记录此数据的前一个数据和后一个数据。
优点：对于频繁的遍历操作，其效率高于HashSet

#### TreeSet
1. 向TreeSet中添加的数据，要求是相同的对象。
2. 两种排序方式：自然排序（实现Compareable接口）和定制排序（Comparator）。
3. 自然排序中，比较两个对象是否相同的标准为：compareTo()返回0，不再是equals()。
4. 定制排序中，比较两个对象是否相同的标准为：compare()返回0。