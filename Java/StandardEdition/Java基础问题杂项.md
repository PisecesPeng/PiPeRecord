## Java基础问题杂项

- [Java基础问题杂项](#java基础问题杂项)
  - [Java是值传递还是引用传递](#java是值传递还是引用传递)
  - ['=='与'equals'的区别](#与equals的区别)
  - [Java常见命令遍历](#java常见命令遍历)
  - [简述'Thread'、'ThreadLocal'与'ThreadLocalMap'的关系](#简述threadthreadlocal与threadlocalmap的关系)
  - [Java四种引用类型](#java四种引用类型)
  - [快速失败(fail-fast)和安全失败(fail-safe)的区别](#快速失败fail-fast和安全失败fail-safe的区别)
  - [Integer的'MIN VALUE'正负结果比较](#integer的min-value正负结果比较)
  - [List foreach循环中, 使用'remove()'会引发什么结果](#list-foreach循环中-使用remove会引发什么结果)

### Java是值传递还是引用传递

> 答案是 : 值传递!
> 1. 基本类型为参数传递时,是传递'**值**'的拷贝,修改拷贝是无法改变到原值的.
> 1. 对象为参数传递时,是将该对象在内存中的'**地址**'拷贝给了参数,可以改变原值,但是本质还是值传递.

---

### '=='与'equals'的区别

1. 一般情况下
> **'=='的作用:**
> - 基本类型: 比较的是'**值**'是否相同.
> - 引用类型: 比较的是'**地址值**'是否相同.
> 
**'equals'的作用:**
> - 引用类型: 默认情况下,比较的是'**地址值**'是否相同(还有'重写'的情况除外).

2. String类的equals方法
> '=='的作用:
> - 比较的是两字符串的**内存地址(堆内存)**的数值是否相等.
> 
'equals'的作用:
> - 比较的是两字符串的**内容**.

---

### Java常见命令遍历

```bash
java  # 运行Java程序
javac  # 编译Java程序
javah  # 生成C++或C的头文件信息
javap  # 反编译器,显示编译类中可以访问的方法和数据
jdb  # 命令行调试环境,既可在本地,也可在与远程的解释器的一次对话中执行
jps  # 虚拟机进程状态工具,仅查找当前用户的Java进程
jconsole  # Java监视和管理控制台
jstat  # 虚拟机统计监视工具
jar  # 主要用于处理jar包
javadoc  # 主要帮助生成帮助文档
jinfo  # 打印出给定的Java进程或核心文件或远程调试服务器的Java配置信息
jmap  # 主要用于打印指定Java进程或核心文件或远程调试服务器的共享对象内存映射或堆内存细节
jstack  # 打印所有的Java线程的堆栈跟踪信息(可选地包括本机帧信息)
```

---

### 简述'Thread'、'ThreadLocal'与'ThreadLocalMap'的关系

> 1. ThreadLocalMap是ThreadLocal的静态内部类,但是两者的实例对象并不存在继承或包裹的关系.
> 2. ThreadLocal像是一个工具类,提供了一系列的方法去操作ThreadLocalMap.
> 3. ThreadLocal很好的隔离了ThreadLocalMap与Thread,防止开发者直接去操作前者.
> 4. ThreadLocal的方法中会去拿当前Thread(Thread.currentThread方法获得)中的**threadLocals**属性(ThreadLocal.ThreadLocalMap类型).
> 5. ThreadLocal的get方法会判断当前线程的**threadLocals**属性是否'**!= null**','**不为null**'则再判断其entry是否'**!= null**',两者任意'**为null**'则都会为其createMap(),ThreadLocal的set方法与其类似.
> 6. ThreadLocal的**createMap()**会为Thread中的threadLocals属性赋予一个新ThreadLocalMap,其Key则为**ThreadLocal本身**,其Value则为**默认值(null)**.

```
小结(从ThreadLocal的get()方法梳理) : 
ThreadLocal在操作ThreadLocalMap时,会去拿Thread.currentThread的'threadLocals'属性进行操作,
若其属性为null,则为其'new ThreadLocalMap(this, initialValue())',其中Key为ThreadLocal本身,Value为该方法返回值(即默认值,return null).
若其属性不为null,则判断该属性'map.getEntry(this)'的Entry是否'!= null',
若entry为null,则与上文一样处理.
若entry不为null,则返回其value即可.
```

---

### Java四种引用类型

> 1. **强引用**  
> Java的默认声明就是强引用(如:Object o = new Object()),这也是其特点之一,强引用可以直接访问目标对象.  
> 强引用所指向的对象在任何时候都不会被系统回收.JVM宁愿抛出OOM异常,也不会回收强引用所指向的对象,  
> 所以就可能导致内存泄漏.  
> 可将强引用赋值为null,这样就符合回收条件,适时的被GC回收了.  

> 2. **软引用**  
> 被其指向的对象不会被垃圾回收器收集(即使该对象没有强引用指向),只有在内存不足时,系统则会回收软引用对象.  
> 系统回收软引用后,若仍没有足够内存,还是会抛出内存溢出异常.  

> 3. **弱引用**  
> 无论内存是否足够,只要JVM开始垃圾回收,弱引用相关联的对象都会被回收.  

> 4. **虚引用**  
> 一个持有虚引用的对象,和没有引用几乎是一样的,随时可能被垃圾回收器回收.  
> 当试图通过虚引用的get()方法取得强引用时,总是会失败.并且,虚引用必须和引用队列一起使用,它的作用在于跟踪垃圾回收过程.  
> 其构造方法必须强制传入ReferenceQueue.  
> 其get()方法仅仅是返回一个null,不管对象有没有被回收,也就是说将永远无法通过虚引用来获取对象.  
> 其对对象而言无感知,也不会影响对象的生命周期.  
> 其可用来作为对象是否存活的监控.  
> 虚引用有一个很重要的用途就是用来做堆外内存的释放,DirectByteBuffer就是通过虚引用来实现堆外内存的释放的.  

```
简述'引用队列' :
引用队列可以与软引用、弱引用以及虚引用一起配合使用,当垃圾回收器准备回收一个对象时,
如果发现它还有引用,那么就会在回收对象之前,把这个引用加入到与之关联的引用队列中去.
程序可以通过判断引用队列中是否已经加入了引用,来判断被引用的对象是否将要被垃圾回收,
这样就可以在对象被回收之前采取一些必要的措施
```

```
简述'虚可达对象': 
如果一个对象与GC Roots之间仅存在虚引用,则称这个对象为虚可达(phantom reachable)对象.
虚引用的作用在于跟踪垃圾回收过程,在对象被收集器回收时收到一个系统通知.
当垃圾回收器准备回收一个对象时,如果发现它还有虚引用,
就会在垃圾回收后,将这个虚引用加入引用队列,在其关联的虚引用出队前,不会彻底销毁该对象.
所以可以通过检查引用队列中是否有相应的虚引用来判断对象是否已经被回收了.
如果一个对象没有强引用和软引用,对于垃圾回收器而言便是可以被清除的,
在清除之前,会调用其finalize方法,如果一个对象已经被调用过finalize方法但是还没有被释放,它就变成了一个虚可达对象.
与软引用和弱引用不同,显式使用虚引用可以阻止对象被清除,
只有在程序中显式或者隐式移除这个虚引用时,这个已经执行过finalize方法的对象才会被清除.
想要显式的移除虚引用的话,只需要将其从引用队列中取出然后扔掉（置为null）即可.
```

```
ps. 
软引用,弱引用都非常适合来保存那些可有可无的缓存数据,当系统内存不足时,这些缓存数据会被回收,不会导致内存溢出.
而当内存资源充足时,这些缓存数据又可以存在相当长的时间,从而起到加速系统的作用.
```

---

### 快速失败(fail-fast)和安全失败(fail-safe)的区别

> **快速失败(fail-fast)**  
> 迭代器在遍历时会直接访问集合中的内容, 并且在遍历过程中使用一个'**modCount**'变量.  
> 集合在被遍历期间如果内容发生变化, 就会改变'**modCount**'的值.  
> 当迭代器遍历下一个元素之前, 都会检测'**modCount**'是否等于'**expectedmodCount**', 是的话就返回遍历,否则抛出异常'ConcurrentModificationException', 并终止遍历.  

ps. 简而言之, **快速失败**(fail-fast)是基于对底层集合做操作, 它受源集合上修改的影响. java.util包下面的所有的集合类都是快速失败的.


> **安全失败(fail-safe)**  
> 迭代时是对源集合的先拷贝再进行遍历, 在遍历时不是直接在源集合内容上访问的.  
> 这样避免了异常, 但同样地, 迭代器在遍历时是不知道源集合发生的修改的.  

ps. 简而言之, **安全失败**(fail-safe)基于对底层集合做拷贝, 它不受源集合上修改的影响. java.util.concurrent包下面的所有的集合类都是安全失败的.  

---

### Integer的'MIN VALUE'正负结果比较

Java用补码表示整数.  
用这种方法, 使得负数的个数与正数不一致.  
从而导致最小负数是没有对应的正数.  

```
例如:
Integer.MIN_VALUE 等于 -2147483648
Integer.MAX_VALUE 等于 +2147483648 - 1
```

尝试对`.MIN_VALUE`取反时，会得到超过`.MAX_VALUE`的值, 因此会发生溢出变为负数.  
这也会引起其他意外情况, 例如 Math.abs(Integer.MIN_VALUE), 还是负值.  

ps. `byte`和`short`没有这种情况.

---

### List foreach循环中, 使用'remove()'会引发什么结果

```java
// '普通'的代码
List list = ...;
for (Object obj : list) {
    list.remove(obj);
}
```

这样删除肯定是不对的.  
正确的删除, 一般可以使用'**Iterator**'或者JDK1.8后的'**removeIf()**'等等..  
这题目中的比较'坑'的地方如下  
**foreach**是语法糖, 其本质是一个'**Iterator**'遍历, 所以要从'**Iterator**'去解释接下来会出现的结果.  

类似上文, 常见的代码如下(稍微修饰了下):  
```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
Iterator itr = list.iterator();
int loop = 1;  // 用来检测循环次数
while (itr.hasNext()) {
    System.out.println(loop++);
    String s = (String) itr.next();
    if ("a".equals(s)) list.remove(s);
}
```

> 以上代码结果是, 不会报错:  
> 由于在判断'**"a".equals(s)**'为true后,  
> '**remove()**'会使得'**--size**'、会使得'**modCount++**',  
> '**next()**'使得'**cursor = i + 1**',  
> 然后再'**hasNext()**'后, '**cursor != size**'就会为false,  
> 所以不会触发'**next()**', 也就不会触发下面代码  

```java
if (modCount != expectedModCount)
    throw new ConcurrentModificationException();
```

> 如果将"a"改为"b", 这块代码就会报错:  
> 遍历到"b"时候, 同样'**remove(**)'会使得'**--size**'、会使得'**modCount++**',  
> 然后再'**hasNext()**'后, '**cursor != size**'就会为true,  
> 之所以为true, 实际是这两个值打了个交叉, cursor增加, size减少, 导致'!='还是成立,  
> 所以会继续触发'**next()**', 以至于触发下面代码, 抛出了异常  

```java
if (modCount != expectedModCount)
    throw new ConcurrentModificationException();
```

ps.  
最上面的例子中, 并没有给出实例化的类,  
我们可以假设其实例化的是线程安全之类(如: CopyOnWriteArrayList ..).  
那结果可能就没问题了.  

