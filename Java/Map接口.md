# Map接口

|----Map：双列数据，存储key-value对的数据
	|----HashMap:作为Map的主要实现类；线程不安全，效率高；可存储null的key和value
		|----LinkedHashMap:保证在遍历map元素时，可以按照添加的顺序实现遍历；原因:在原有的HashMap底层结构基础上，增加一对指针，指向前一个和后一个元素；
	!----TreeMap:保证按照添加的key-value对进行排序，实现排序遍历，此时考虑key的自然排序或者定制排序，底层使用红黑树；
	|----Hashtable:作为古老的实现类：线程安全，效率低；不可存储null的key和value
		|----Properties:常用来处理配置文件。key和value都是string类型
HashMap的底层：数组加链表（JDK7及之前）数组加链表加红黑树（JDK8）
面试题：
1. HashMap的底层实现原理？
2. HashMap和Hashtable的异同？
3. CurrentHashMap与Hashtable的异同？

## Map结构的理解
- Map中的key：无序的、不可重复的、使用set存储所有的key --->key所在的类要重写equals()和hashCode()
- Map中的value：无序的、可重复的，使用Collection存储所有的value--->value所在的类要重写equals()
- 一个键值对：key-value构成了一个Entry对现象。
- Map中的entry：无序的、不可重复的，使用Set存储所有的entry

## HashMap的底层实现原理？以jdk7为例说明

```
HashMap map = new HashMap();
```
在实例化之后，底层创建了一个长度为16的一维数组Entry[] table
```
map.put(key1,value1);
```
首先，调用key1所在类的hashCode()计算key1的哈希值，此哈希值经过某种算法以后，得到在Entry数组中的存放位置。
如果此位置上数据为空，此时键值对添加成功。
如果此位置上的数据不为空（意味着此位置上存在一个或者多个数据（以链表形式存在）），比较key1和已经存在的一个或者多个数据的哈希值：

- 如果key1的哈希值与已经存在的哈希值都不相同，此时键值对添加成功。---情况2
- 如果key1的哈希值和已经存在的某一个数据的哈希值相同，继续比较：调用key1的equals()方法，比较：
	- 如果equals()返回false：此时key1-value1添加成功 ---情况3
	- 如果euqals()返回true：使用value1替换value2
补充：关于情况2和情况3：此时key1-和val1和原来的数据以链表方式储存。
在不断的添加过程中，会涉及到扩容问题，默认扩容方式：扩容容量为原来的两倍，并将原来的数据复制过来。

jdk8相较于jdk7底层实现的方面不同；
1. new HashMap()：底层没有创建一个长度为16的数组
2. jdk8底层的数组是：Node[]而非Entry[]
3. 首先调用put()方法时，底层创建长度为16的数组
4. jdk7底层结构只有：数组+链表。jdk8底层结构：数组+链表+红黑树。
5. 当数组的某一个索引位置上的元素以链表形式存在的数据个数>8且当前数组长度>64时，此时此索引位置上的所有数据改为红黑树储存。

DEFAULT_INITIAL_CAPACITY：HashMap的默认容量：16
DEFAULT_LOAD_FACTOR：HashMap的默认加载因子：0.75
threshold：扩容临界值：容量*填充因子：16*0.75=12
TREEIFY_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树：8
MIN_TREEIFY_CAPACITY：桶中的Node被树化时最小的hash表容量：64

## Map中定义的方法
添加、删除、修改操作
```java
Object put(Object key,Object value);//将指定key-value添加或修改到当前map对象中
void putAll(Map m);//将m中所有的键值对都存放到map中
Object remove(Object key);//移除指定的key的键值对
void clear();//清空当前map中所有数据
```
元素查询操作
```java
Object get(Object key);//获取指定key对应的value
boolean containsKey(Object key);
boolean containsValue(Object value);
int size();
boolean isEmpty();
boolean equals(Object obj);//判断当前map和参数对象obj是否相等
```
元视图操作方法
```java
Set keySet();//返回所有key对应的value
Collection values();//返回所有value构成的Collection集合
Set entrySet();//返回所有key-value对构成的Set集合
```
## Collections：操作Collection，Map的工具类
面试题：Collections和Collection的区别
```java
reverse(List);//反转List中的元素的顺序
shuffle(List);//对List集合元素进行随机排序
sort(List);//根据元素的自然顺序对指定List集合元素按升序排序
sort(List,Comparator);//根据指定的Comparator产生的顺序对List集合元素进行排序
swap(List,int,int);//将指定List集合中的i处元素和j处元素进行交换
Object max(Collection);//根据元素的自然排序，返回给定集合中的最大元素
Object max(Collection,comparator);//根据Comparator指定的顺序，返回给定集合中最大的值
Object min(Collection);
Object min(Collection,comparator);
int frequency(Collection,Object);//返回指定集合中指定元素的出现次数
void copy(List dest,List src);//将src中的内容复制到dest中
boolean replaceAll(List list,Object oldVal,Object newVal);//使用新值替换List中旧值；
```
Collection类中提供了多个**synchronizedxx()**方法，该方法可使指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题