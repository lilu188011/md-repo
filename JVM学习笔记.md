## 													JVM学习笔记

### 一、前言

#### 1.1 JDK8

```
官网 :https://docs.oracle.com/javase/8/
```

#### 1.2  The relation of JDK/JRE/JVM

Reference -> Developer Guides -> 定位到:https://docs.oracle.com/javase/8/docs/index.html

```
Oracle has two products that implement Java Platform Standard Edition (Java SE)
8: Java SE Development Kit (JDK) 8 and Java SE Runtime Environment (JRE) 8.
JDK 8 is a superset of JRE 8, and contains everything that is in JRE 8, plus
tools such as the compilers and debuggers necessary for developing applets and
applications. JRE 8 provides the libraries, the Java Virtual Machine (JVM), and
other components to run applets and applications written in the Java programming
language. Note that the JRE includes components not required by the Java SE
specification, including both standard and non-standard Java components.
```

![image-20220718160027854](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718160027854.png)

### 二、 源码到文件

#### 2.1 源码

```java
class Person{
  private String name;
  private int age;
  private static String address;
  private final static String hobby="Programming";
  public void say(){
    System.out.println("person say...");
 }
  public int calc(int op1,int op2){
    return op1+op2;
 }
}
```

**编译: javac Person.java ---> Person.class**

#### 2.2 编译过程

```
Person.java -> 词法分析器 -> tokens流 -> 语法分析器 -> 语法树/抽象语法树 -> 语义分析器
-> 注解抽象语法树 -> 字节码生成器 -> Person.class文件
```

#### 2.3 类文件(Class文件)

官网The class File Format :https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html

```
cafe babe 0000 0034 0027 0a00 0600 1809
0019 001a 0800 1b0a 001c 001d 0700 1e07
001f 0100 046e 616d 6501 0012 4c6a 6176
612f 6c61 6e67 2f53 7472 696e 673b 0100
0361 6765 0100 0149 0100 0761 6464 7265
......
```

> magic(魔数):
>
> > The  magic  item supplies the magic number identifying the  class  file format; it has the
> > value  0xCAFEBABE .

```
cafe babe
```

> minor_version, major_version

```
0000 0034  对应10进制的52，代表JDK 8中的一个版本
```

> constant_pool_count

```
0027  对应十进制27，代表常量池中27个常量
```

```
ClassFile {
 u4       magic;
 u2       minor_version;
 u2       major_version;
 u2       constant_pool_count;
 cp_info    constant_pool[constant_pool_count-1];
 u2       access_flags;
 u2       this_class;
 u2       super_class;
 u2       interfaces_count;
 u2       interfaces[interfaces_count];
 u2       fields_count;
 field_info   fields[fields_count];
 u2       methods_count;
 method_info  methods[methods_count];
 u2       attributes_count;
 attribute_info attributes[attributes_count];
}
```

> .class字节码文件
>
> ```
> 魔数与class文件版本
> 常量池
> 访问标志
> 类索引、父类索引、接口索引
> 字段表集合
> 方法表集合
> 属性表集合
> ```

### 三、类文件到虚拟机(类加载机制)

#### 3.1 转载（Load)

> 查找和导入class文件

```
(1)通过一个类的全限定名获取定义此类的二进制字节流
(2)将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
(3)在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口
```

#### 3.2 链接(Link)

##### 3.2.1  验证(Verify)

> 保证被加载类的正确性

- 文件格式验证
- 元数据验证
- 字节码验证
- 符号引用验证

##### 3.2.2 准备(Prepare)

> 为类的静态变量分配内存，并将其初始化为默认值

##### 3.2.3 解析(Resolve)

> 把类中的符号引用转换为直接引用

#### 3.3 初始化(Initialize)

> 对类的静态变量，静态代码块执行初始化操作

#### 3.4 类加载机制图解

> 使用和卸载不算是类加载过程中的阶段，只是画完整了一下

![image-20220718161242614](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718161242614.png)

### 四、类加载器

> 在装载(Load)阶段，其中第(1)步:通过类的全限定名获取其定义的二进制字节流，需要借助类装载
> 器完成，顾名思义，就是用来装载Class文件的。
> (1)通过一个类的全限定名获取定义此类的二进制字节流

#### 4.1 分类

```
1）Bootstrap ClassLoader 负责加载$JAVA_HOME中 jre/lib/rt.jar 里所有的class或 
												Xbootclassoath选项指定的jar包。由C++实现，不是ClassLoader子类。
2）Extension ClassLoader 负责加载java平台中扩展功能的一些jar包，
												包括$JAVA_HOME中jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。
3）App ClassLoader 	负责加载classpath中指定的jar包及 Djava.class.path 所指定目录下的类和jar包。

4）Custom ClassLoader 通过java.lang.ClassLoader的子类自定义加载class，属于应用程序根据
自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader。
```

#### 4.2 图解

![image-20220718161619937](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718161619937.png)

#### 4.3 加载原则

```
检查某个类是否已经加载：顺序是自底向上，从Custom ClassLoader到BootStrap ClassLoader逐层检
查，只要某个Classloader已加载，就视为已加载此类，保证此类只所有ClassLoader加载一次。
加载的顺序：加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。
```

> 双亲委派机制
>
> > 定义：如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把
> > 这个请求任务委托给父类加载器去完成，依次递归，如果父类加载器可以完成类加载任务，就
> > 成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。
> > 优势：Java类随着加载它的类加载器一起具备了一种带有优先级的层次关系。比如，Java中的
> > Object类，它存放在rt.jar之中,无论哪一个类加载器要加载这个类，最终都是委派给处于模型
> > 最顶端的启动类加载器进行加载，因此Object在各种类加载环境中都是同一个类。如果不采用
> > 双亲委派模型，那么由各个类加载器自己取加载的话，那么系统中会存在多种不同的Object
> > 类。
> > 破坏：可以继承ClassLoader类，然后重写其中的loadClass方法，其他方式大家可以自己了解
> > 拓展一下。
>
> 

### 五、运行时数据区

> 在装载阶段的第(2),(3)步可以发现有运行时数据，堆，方法区等名词
> (2)将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
> (3)在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口

> 说白了就是类文件被类装载器装载进来之后，类中的内容(比如变量，常量，方法，对象等这些数
> 据得要有个去处，也就是要存储起来，存储的位置肯定是在JVM中有对应的空间)

#### 5.1 官网概括

https://docs.oracle.com/javase/specs/jvms/se8/html/index.html

> Summary

```
The Java Virtual Machine defines various run-time data areas that are used
during execution of a program. Some of these data areas are created on Java
Virtual Machine start-up and are destroyed only when the Java Virtual Machine
exits. Other data areas are per thread. Per-thread data areas are created when a
thread is created and destroyed when the thread exits.
```

#### 5.2 图解

![image-20220718162103956](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718162103956.png)

#### 5.3 常规理解

##### 5.3.1 Method Area(方法区)

方法区是各个线程共享的内存区域，在虚拟机启动时创建。
用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却又一个别名叫做Non-Heap(非堆)，目
的是与Java堆区分开来。
当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

```
The Java Virtual Machine has a method area that is shared among all Java Virtual
Machine threads.
The method area is created on virtual machine start-up.
Although the method area is logically part of the heap,......
If memory in the method area cannot be made available to satisfy an allocation
request, the Java Virtual Machine throws an OutOfMemoryError.
```

> 此时回看装载阶段的第2步：(2)将这个字节流所代表的静态存储结构转化为方法区的运行时数据
> 结构

如果这时候把从Class文件到装载的第(1)和(2)步合并起来理解的话，可以画个图

![image-20220718162256116](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718162256116.png)

> 值得说明的

(1)方法区在JDK 8中就是Metaspace，在JDK6或7中就是Perm Space
(2)Run-Time Constant Pool

Class文件中除了有类的版本、字段、方法、接口等描述
信息外，还有一项信息就是常量池，用于存放编译时期生成的各种字面量和符号引用，这部分内容将在
类加载后进
入方法区的运行时常量池中存放。

```
Each run-time constant pool is allocated from the Java Virtual Machine's method
area (§2.5.4).s
```

##### 5.3.2 Heap(堆)

Java堆是Java虚拟机所管理内存中最大的一块，在虚拟机启动时创建，被所有线程共享。
Java对象实例以及数组都在堆上分配。

```
The Java Virtual Machine has a heap that is shared among all Java Virtual
Machine threads. The heap is the run-time data area from which memory for all
class instances and arrays is allocated.
The heap is created on virtual machine start-up.
```

> 此时回看装载阶段的第3步：(3)在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方
> 法区中这些数据的访问入口

此时装载(1)(2)(3)的图可以改动一下

![image-20220718162434076](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718162434076.png)

##### 5.3.3 Java Virtual Machine Stacks(虚拟机栈)

> 经过上面的分析，类加载机制的装载过程已经完成，后续的链接，初始化也会相应的生效。
> 假如目前的阶段是初始化完成了，后续做啥呢？肯定是Use使用咯，不用的话这样折腾来折腾去
> 有什么意义？那怎样才能被使用到？换句话说里面内容怎样才能被执行？比如通过主函数main调
> 用其他方法，这种方式实际上是main线程执行之后调用的方法，即要想使用里面的各种内容，得
> 要以线程为单位，执行相应的方法才行。
> 那一个线程执行的状态如何维护？一个线程可以执行多少个方法？这样的关系怎么维护呢？

虚拟机栈是一个线程执行的区域，保存着一个线程中方法的调用状态。换句话说，一个Java线程的运行
状态，由一个虚拟机栈来保存，所以虚拟机栈肯定是线程私有的，独有的，随着线程的创建而创建。
每一个被线程执行的方法，为该栈中的栈帧，即每个方法对应一个栈帧。
调用一个方法，就会向栈中压入一个栈帧；一个方法调用完成，就会把该栈帧从栈中弹出。

```
Each Java Virtual Machine thread has a private Java Virtual Machine stack,
created at the same time as the thread. A Java Virtual Machine stack stores
frames (§2.6).
```

> 画图理解栈和栈帧

![image-20220718162538743](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718162538743.png)

##### 5.3.4 The pc Register(程序计数器)

> 我们都知道一个JVM进程中有多个线程在执行，而线程中的内容是否能够拥有执行权，是根据
> CPU调度来的。
> 假如线程A正在执行到某个地方，突然失去了CPU的执行权，切换到线程B了，然后当线程A再获
> 得CPU执行权的时候，怎么能继续执行呢？这就是需要在线程中维护一个变量，记录线程执行到
> 的位置。

程序计数器占用的内存空间很小，由于Java虚拟机的多线程是通过线程轮流切换，并分配处理器执行时
间的方式来实现的，在任意时刻，一个处理器只会执行一条线程中的指令。因此，为了线程切换后能够
恢复到正确的执行位置，每条线程需要有一个独立的程序计数器(线程私有)。
如果线程正在执行Java方法，则计数器记录的是正在执行的虚拟机字节码指令的地址；
如果正在执行的是Native方法，则这个计数器为空。

```
The Java Virtual Machine can support many threads of execution at once (JLS
§17). Each Java Virtual Machine thread has its own pc (program counter)
register. At any point, each Java Virtual Machine thread is executing the code
of a single method, namely the current method (§2.6) for that thread. If that
method is not native, the pc register contains the address of the Java Virtual
Machine instruction currently being executed. If the method currently being
executed by the thread is native, the value of the Java Virtual Machine's pc
register is undefined. The Java Virtual Machine's pc register is wide enough to
hold a returnAddress or a native pointer on the specific platform.
```

##### 5.3.5 Native Method Stacks(本地方法栈)

如果当前线程执行的方法是Native类型的，这些方法就会在本地方法栈中执行。

### 六、 结合字节码指令理解Java虚拟机栈和栈帧

> 官网 ：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6
> 栈帧：每个栈帧对应一个被调用的方法，可以理解为一个方法的运行空间

每个栈帧中包括局部变量表(Local Variables)、操作数栈(Operand Stack)、指向运行时常量池的引用(A reference to
the run-time constant pool)、方法返回地址(Return Address)和附加信息

```
局部变量表:方法中定义的局部变量以及方法的参数存放在这张表中
局部变量表中的变量不可直接使用，如需要使用的话，必须通过相关指令将其加载至操作数栈中作为操作数使用。
```

```
操作数栈:以压栈和出栈的方式存储操作数的
```

```
动态链接:每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态
连接(Dynamic Linking)。
```

```
方法返回地址:当一个方法开始执行后,只有两种方式可以退出，一种是遇到方法返回的字节码指令；一种是遇见异常，并且
这个异常没有在方法体内得到处理。
```

![image-20220718162907887](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718162907887.png)

```java
class Person{
  private String name="Jack";
  private int age;
private final double salary=100;
  private static String address;
  private final static String hobby="Programming";
  public void say(){
    System.out.println("person say...");
 }
  public static int calc(int op1,int op2){
    op1=3;
    int result=op1+op2;
    return result;
 }
  public static void order(){
   
 }
public static void main(String[] args){
calc(1,2);
    order();
}
}
```

> 此时你需要一个能够看懂反编译指令的宝典
> 比如官网的：https://docs.oracle.com/javase/specs/jvms/se8/html/index.html
> 当然网上也有很多，大家可以自己查找

```java
Compiled from "Person.java"
class Person {
...     
 public static int calc(int, int);
  Code:
   0: iconst_3   //将int类型常量3压入[操作数栈]
   1: istore_0   //将int类型值存入[局部变量0]
   2: iload_0    //从[局部变量0]中装载int类型值入栈
   3: iload_1    //从[局部变量1]中装载int类型值入栈
   4: iadd     //将栈顶元素弹出栈，执行int类型的加法，结果入栈
   【For example, the iadd instruction (§iadd) adds two int values together. It
requires that the int values to be added be the top two values of the operand stack, pushed
there by previous instructions. Both of the int values are popped from the operand stack.
They are added, and their sum is pushed back onto the operand stack. Subcomputations may be
nested on the operand stack, resulting in values that can be used by the encompassing
computation.】
   5: istore_2   //将栈顶int类型值保存到[局部变量2]中
   6: iload_2    //从[局部变量2]中装载int类型值入栈
   7: ireturn    //从方法中返回int类型的数据
...
}
```

![image-20220718163037032](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718163037032.png)

#### 折腾一下

##### 栈指向堆

如果在栈帧中有一个变量，类型为引用类型，比如Object obj=new Object()，这时候就是典型的栈中元素指向堆中的
对象。

![image-20220718163151249](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718163151249.png)

##### 方法区指向堆

方法区中会存放静态变量，常量等数据。如果是下面这种情况，就是典型的方法区中元素指向堆中的对象。

```
private static Object obj=new Object();
```

![image-20220718163246333](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718163246333.png)

##### 堆指向方法区

方法区中会包含类的信息，堆中会有对象，那怎么知道对象是哪个类创建的呢？

![image-20220718163435931](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718163435931.png)

**思考** ：一个对象怎么知道它是由哪个类创建出来的？怎么记录？这就需要了解一个Java对象的具体信息咯。

### 七、Java对象内存布局

> 一个Java对象在内存中包括3个部分：对象头、实例数据和对齐填充

![image-20220718163531684](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718163531684.png)

### 八、 内存模型

#### 8.1 图解

```
一块是非堆区，一块是堆区。
堆区分为两大块，一个是Old区，一个是Young区。
Young区分为两大块，一个是Survivor区（S0+S1），一块是Eden区。 Eden:S0:S1=8:1:1
S0和S1一样大，也可以叫From和To。
```

![image-20220718163649404](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718163649404.png)

> 根据之前对于Heap的介绍可以知道，一般对象和数组的创建会在堆中分配内存空间，关键是堆中有这么多区
> 域，那一个对象的创建到底在哪个区域呢？

#### 8.2 对象创建所在区域

一般情况下，新创建的对象都会被分配到Eden区，一些特殊的大的对象会直接分配到Old区

> 比如有对象A，B，C等创建在Eden区，但是Eden区的内存空间肯定有限，比如有100M，假如已经使用了
> 100M或者达到一个设定的临界值，这时候就需要对Eden内存空间进行清理，即垃圾收集(Garbage Collect)，
> 这样的GC我们称之为Minor GC，Minor GC指得是Young区的GC。
> 经过GC之后，有些对象就会被清理掉，有些对象可能还存活着，对于存活着的对象需要将其复制到Survivor
> 区，然后再清空Eden区中的这些对象。

#### 8.3 Survivor区详解

由图解可以看出，Survivor区分为两块S0和S1，也可以叫做From和To。
在同一个时间点上，S0和S1只能有一个区有数据，另外一个是空的。

> 接着上面的GC来说，比如一开始只有Eden区和From中有对象，To中是空的。
> 此时进行一次GC操作，From区中对象的年龄就会+1，我们知道Eden区中所有存活的对象会被复制到To区，
> From区中还能存活的对象会有两个去处。
> 若对象年龄达到之前设置好的年龄阈值，此时对象会被移动到Old区， 如果Eden区和From区 没有达到阈值的
> 对象会被复制到To区。 此时Eden区和From区已经被清空(被GC的对象肯定没了，没有被GC的对象都有了各
> 自的去处)。
> 这时候From和To交换角色，之前的From变成了To，之前的To变成了From。
> 也就是说无论如何都要保证名为To的Survivor区域是空的。

Minor GC会一直重复这样的过程，知道To区被填满，然后会将所有对象复制到老年代中。

#### 8.4 Old区详解

从上面的分析可以看出，一般Old区都是年龄比较大的对象，或者相对超过了某个阈值的对象。
在Old区也会有GC的操作，Old区的GC我们称作为Major GC 。

#### 8.5 对象的一辈子理解

```
我是一个普通的Java对象,我出生在Eden区,在Eden区我还看到和我长的很像的小兄弟,我们在Eden区中玩了挺长时间。有
一天Eden区中的人实在是太多了,我就被迫去了Survivor区的“From”区,自从去了Survivor区,我就开始漂了,有时候在
Survivor的“From”区,有时候在Survivor的“To”区,居无定所。直到我18岁的时候,爸爸说我成人了,该去社会上闯闯
了。
于是我就去了年老代那边,年老代里,人很多,并且年龄都挺大的,我在这里也认识了很多人。在年老代里,我生活了20年(每次
GC加一岁),然后被回收。
```

![image-20220718164032825](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718164032825.png)

#### 8.6 空间担保机制

```java
空间担保机制：
if 年轻代总大小 > 老年代连续可用空间
    Gc是安全的    MinorGC  RunIng
if 年轻代总大小 < 老年代连续可用空间
    Gc是不是安全的，检查是否开启了允许担保失败
    if：开启了：
        检查历次GC到老年代的数据大小平均值
        若历次的平均值 < 当前老年代连续可用空间大小，MinorGC有风险，执行MinorGC
    if：没有开启 || 若历次的平均值 >  当前老年代连续可用空间大小 
            FullGC
```

### 九、垃圾回收

#### 9.1 如何确定一个对象是垃圾？

> 要想进行垃圾回收，得先知道什么样的对象是垃圾

##### 9.1.1  引用计数法

对于某个对象而言，只要应用程序中持有该对象的引用，就说明该对象不是垃圾，如果一个对象没有任
何指针对其
引用，它就是垃圾

**弊端** :如果AB相互持有引用，导致永远不能被回收。

##### 9.1.2 可达性分析

通过GC Root的对象，开始向下寻找，看某个对象是否可达

> 能作为GC Root:类加载器、Thread、虚拟机栈的本地变量表、static成员、常量引用、本地方法
> 栈的变量等。

#### 9.2  垃圾收集算法

> 已经能够确定一个对象为垃圾之后，接下来要考虑的就是回收，怎么回收呢？
> 得要有对应的算法，下面聊聊常见的垃圾回收算法。

##### 9.2.1 标记-清除(Mark-Sweep)

- 标记

找出内存中需要回收的对象，并且把它们标记出来

> 此时堆中所有的对象都会被扫描一遍，从而才能确定需要回收的对象，比较耗时

![image-20220718165120490](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165120490.png)

- 清除

  清除掉被标记需要回收的对象，释放出对应的内存空间

![image-20220718165145204](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165145204.png)

缺点

```
标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程
序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
(1)标记和清除两个过程都比较耗时，效率不高
(2)会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无
法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
```

##### 9.2.2 复制(Copying)

将内存划分为两块相等的区域，每次只使用其中一块，如下图所示：

![image-20220718165242795](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165242795.png)

当其中一块内存使用完了，就将还存活的对象复制到另外一块上面，然后把已经使用过的内存空间一次
清除掉。

![image-20220718165253869](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165253869.png)

缺点: 空间利用率降低。

##### 9.2.3 标记-整理(Mark-Compact)

标记过程仍然与"标记-清除"算法一样，但是后续步骤不是直接对可回收对象进行清理，而是让所有存活
的对象都向一端移动，然后直接清理掉端边界以外的内存。

![image-20220718165334314](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165334314.png)

让所有存活的对象都向一端移动，清理掉边界意外的内存。让所有存活的对象都向一端移动，清理掉边界意外的内存。

![image-20220718165343093](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165343093.png)

#### 9.3  分代收集算法

> 既然上面介绍了3中垃圾收集算法，那么在堆内存中到底用哪一个呢？

Young区：复制算法(对象在被分配之后，可能生命周期比较短，Young区复制效率比较高)
Old区：标记清除或标记整理(Old区对象存活时间比较长，复制来复制去没必要，不如做个标记再清理)

#### 9.4 垃圾收集器

> 如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现，说白了就是落
> 地咯。

![image-20220718165452034](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165452034.png)

##### 9.4.1 Serial收集器

Serial收集器是最基本、发展历史最悠久的收集器，曾经（在JDK1.3.1之前）是虚拟机新生代收集的唯
一选择。
它是一种单线程收集器，不仅仅意味着它只会使用一个CPU或者一条收集线程去完成垃圾收集工作，更
重要的是其在进行垃圾收集的时候需要暂停其他线程。

```
优点：简单高效，拥有很高的单线程收集效率
缺点：收集过程需要暂停所有线程
算法：复制算法
适用范围：新生代
应用：Client模式下的默认新生代收集器
```

![image-20220718165542006](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165542006.png)

##### 9.4.2 ParNew收集器

可以把这个收集器理解为Serial收集器的多线程版本。

```
优点：在多CPU时，比Serial效率高。
缺点：收集过程暂停所有应用程序线程，单CPU时比Serial效率差。
算法：复制算法
适用范围：新生代
应用：运行在Server模式下的虚拟机中首选的新生代收集器
```

![image-20220718165611597](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165611597.png)

##### 9.4.3 Parallel Scavenge收集器

Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集
器，看上去和ParNew一样，但是Parallel Scanvenge更关注 **系统的吞吐量** 。

> ，看上去和ParNew一样，但是Parallel Scanvenge更关注 系统的吞吐量 。
> 吞吐量=运行用户代码的时间/(运行用户代码的时间+垃圾收集时间)
> 比如虚拟机总共运行了100分钟，垃圾收集时间用了1分钟，吞吐量=(100-1)/100=99%。
> 若吞吐量越大，意味着垃圾收集的时间越短，则用户代码可以充分利用CPU资源，尽快完成程序
> 的运算任务。

```
-XX:MaxGCPauseMillis控制最大的垃圾收集停顿时间，
-XX:GC Time Ratio直接设置吞吐量的大小。
```

##### 9.4.4 Serial Old收集器

Serial Old收集器是Serial收集器的老年代版本，也是一个单线程收集器，不同的是采用"标记-整理算
法"，运行过程和Serial收集器一样

![image-20220718165742072](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165742072.png)

##### 9..4.5 Parallel Old收集器

Parallel Old收集器是Parallel Scavenge收集器的老年代版本，使用多线程和"标记-整理算法"进行垃圾
回收。
**吞吐量优先**

##### 9.4.6 CMS收集器

CMS(Concurrent Mark Sweep)收集器是一种以获取 最短回收停顿时间 为目标的收集器。
采用的是"标记-清除算法",整个过程分为4步

```
(1)初始标记 CMS initial mark      标记GC Roots能关联到的对象 Stop The World--
->速度很快
(2)并发标记 CMS concurrent mark    进行GC Roots Tracing
(3)重新标记 CMS remark         修改并发标记因用户程序变动的内容 Stop The
World
(4)并发清除 CMS concurrent sweep
```

> 由于整个过程中，并发标记和并发清除，收集器线程可以与用户线程一起工作，所以总体上来
> 说，CMS收集器的内存回收过程是与用户线程一起并发地执行的。

```
优点：并发收集、低停顿
缺点：产生大量空间碎片、并发阶段会降低吞吐量
```

![image-20220718165936183](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718165936183.png)

##### 9.4.7 G1收集器

G1特点

```
并行与并发
分代收集（仍然保留了分代的概念）
空间整合（整体上属于“标记-整理”算法，不会导致空间碎片）
可预测的停顿（比CMS更先进的地方在于能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集
上的时间不得超过N毫秒）
```

> 使用G1收集器时，Java堆的内存布局与就与其他收集器有很大差别，它将整个Java堆划分为多个
> 大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再
> 是物理隔离的了，它们都是一部分Region（不需要连续）的集合

**工作过程可以分为如下几步**

```
初始标记（Initial Marking）   标记一下GC Roots能够关联的对象，并且修改TAMS的值，需要暂
停用户线程
并发标记（Concurrent Marking）  从GC Roots进行可达性分析，找出存活的对象，与用户线程并发
执行
最终标记（Final Marking）    修正在并发标记阶段因为用户程序的并发执行导致变动的数据，需
暂停用户线程
筛选回收（Live Data Counting and Evacuation） 对各个Region的回收价值和成本进行排序，根据
用户所期望的GC停顿时间制定回收计划
```

![image-20220718170116384](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718170116384.png)

##### 9..4.8 垃圾收集器分类

- 串行收集器->Serial和Serial Old

  只能有一个垃圾回收线程执行，用户线程暂停。 适用于内存比较小的嵌入式设备 。

- 并行收集器[吞吐量优先]->Parallel Scanvenge、Parallel Old

  多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。 适用于科学计算、后台处理等若交互场
  景 

- 并发收集器[停顿时间优先]->CMS、G1

  用户线程和垃圾收集线程同时执行(但并不一定是并行的，可能是交替执行的)，垃圾收集线程在执行的
  时候不会停顿用户线程的运行。 适用于相对时间有要求的场景，比如Web 。

##### 9.4.9 理解吞吐量和停顿时间

- 停顿时间->垃圾收集器  进行 垃圾回收终端应用执行响应的时间
- 吞吐量->运行用户代码时间/(运行用户代码时间+垃圾收集时间)

```
停顿时间越短就越适合需要和用户交互的程序，良好的响应速度能提升用户体验；
高吞吐量则可以高效地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任
务。
```

小结 :这两个指标也是评价垃圾回收器好处的标准，其实调优也就是在观察者两个变量。

##### 9.4.10  如何选择合适的垃圾收集器

官网 ：https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html#sthref28

- 优先调整堆的大小让服务器自己来选择
- 如果内存小于100M，使用串行收集器
- 如果是单核，并且没有停顿时间要求，使用串行或JVM自己选
- 如果允许停顿时间超过1秒，选择并行或JVM自己选
- 如果响应时间最重要，并且不能超过1秒，使用并发收集器
- 对于G1收集

##### 9.4.11 再次理解G1

> JDK 7开始使用，JDK 8非常成熟，JDK 9默认的垃圾收集器，适用于新老生代。

判断是否需要使用G1收集器？

```
（1）50%以上的堆被存活对象占用
（2）对象分配和晋升的速度变化非常大
（3）垃圾回收时间比较长
```

##### 9.4.12 如何开启需要的垃圾收集器

```
（1）串行
-XX：+UseSerialGC
-XX：+UseSerialOldGC
（2）并行(吞吐量优先)：
 -XX：+UseParallelGC
 -XX：+UseParallelOldGC
（3）并发收集器(响应时间优先)
-XX：+UseConcMarkSweepGC
-XX：+UseG1GC
```

### 十、JVM参数

#### 10.1.1 标准参数

```
-version
-help
-server
-cp
```

#### 10.1.2 -X参数

> 非标准参数，也就是在JDK各个版本中可能会变动

```
-Xint   解释执行
-Xcomp  第一次使用就编译成本地代码
-Xmixed  混合模式，JVM自己来决定
```

#### 10.1.3 -XX参数

> **使用得最多的参数类型**
> 非标准化参数，相对不稳定，主要用于JVM调优和Debug

```
a.Boolean类型
格式：-XX:[+-]<name>      +或-表示启用或者禁用name属性
比如：-XX:+UseConcMarkSweepGC  表示启用CMS类型的垃圾回收器
-XX:+UseG1GC       表示启用G1类型的垃圾回收器
b.非Boolean类型
格式：-XX<name>=<value>表示name属性的值是value
比如：-XX:MaxGCPauseMillis=500
```

#### 10.1.4 其他参数

```
-Xms1000等价于-XX:InitialHeapSize=1000
-Xmx1000等价于-XX:MaxHeapSize=1000
-Xss100等价于-XX:ThreadStackSize=100
```

#### 10.1.5 查看参数

> java -XX:+PrintFlagsFinal -version > flags.txt

> 值得注意的是"="表示默认值，":="表示被用户或JVM修改后的值
> 要想查看某个进程具体参数的值，可以使用jinfo，这块后面聊
> 一般要设置参数，可以先查看一下当前参数是什么，然后进行修改

```
(1)设置堆内存大小和参数打印
-Xmx100M -Xms100M -XX:+PrintFlagsFinal
(2)查询+PrintFlagsFinal的值
:=true
(3)查询堆内存大小MaxHeapSize
:= 104857600
(4)换算
104857600(Byte)/1024=102400(KB)
102400(KB)/1024=100(MB)
(5)结论
104857600是字节单位
```

#### 10.1.6 常用参数含义

| 参数                                                         | 含义                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -XX:CICompilerCount=3                                        | 最大并行编译数                                               | 如果设置大于1，虽然编译速度会提高，但是同样影响系<br/>统稳定性，会增加JVM崩溃的可能 |
| -XX:InitialHeapSize=100M                                     | 初始化堆大小                                                 | 简写-Xms100M                                                 |
| -XX:MaxHeapSize=100M                                         | 最大堆大小                                                   | 简写-Xmx 100M                                                |
| -XX:NewSize=20M                                              | 设置年轻代的大小                                             |                                                              |
| -XX:MaxNewSize=50M                                           | 年轻代最大大小                                               |                                                              |
| -XX:OldSize=50M                                              | 设置老年代大小                                               |                                                              |
| -XX:MetaspaceSize=50M                                        | 设置方法区大小                                               |                                                              |
| -XX:MaxMetaspaceSize=50M                                     | 方法区最大大小                                               |                                                              |
| -XX:+UseParallelGC                                           | 使用UseParallelGC                                            | 新生代，吞吐量优先                                           |
| -XX:+UseParallelOldGC                                        | 使用UseParallelOldGC                                         | 老年代，吞吐量优先                                           |
| -XX:+UseConcMarkSweepGC                                      | 使用CMS                                                      | 老年代，停顿时间优先                                         |
| -XX:+UseG1GC                                                 | 使用G1GC                                                     | 新生代，老年代，停顿时间优先                                 |
| -XX:NewRatio                                                 | 新老生代的比值                                               | 比如-XX:Ratio=4，则表示新生代:老年代=1:4，也就是新<br/>生代占整个堆内存的1/5 |
| -XX:SurvivorRatio                                            | 两个S区和Eden区的比值                                        | 比如-XX:SurvivorRatio=8，也就是(S0+S1):Eden                  |
| -XX:+HeapDumpOnOutOfMemoryErro                               | 启动堆内存溢出打印                                           | 当JVM堆内存发生溢出时，也就是OOM，自动生成dump<br/>文件      |
| -XX:HeapDumpPath=heap.hprof                                  | 指定堆内存溢出打印目录                                       | 表示在当前目录生成一个heap.hprof文件                         |
| XX:+PrintGCDetails -<br/>XX:+PrintGCTimeStamps -<br/>XX:+PrintGCDateStamps<br/>Xloggc:$CATALINA_HOME/logs/gc.log | 打印出GC日志                                                 | 可以使用不同的垃圾收集器，对比查看GC情况                     |
| -Xss128k                                                     | 设置每个线程的堆栈大小                                       | 经验值是3000-5000最佳                                        |
| -XX:MaxTenuringThreshold=6                                   | 提升年老代的最大临界值                                       | 默认值为 15                                                  |
| -XX:InitiatingHeapOccupancyPercent                           | 启动并发GC周期时堆内存使用占比                               | G1之类的垃圾收集器用它来触发并发GC周期,基于整个堆<br/>的使用率,而不只是某一代内存的使用比. 值为 0 则表<br/>示”一直执行GC循环”. 默认值为 45. |
| -XX:G1HeapWastePercent                                       | 允许的浪费堆空间的占比                                       | 默认是10%，如果并发标记可回收的空间小于10%,则不<br/>会触发MixedGC。 |
| -XX:MaxGCPauseMillis=200ms                                   | G1最大停顿时间                                               | 暂停时间不能太小，太小的话就会导致出现G1跟不上垃<br/>圾产生的速度。最终退化成Full GC。所以对这个参数的<br/>调优是一个持续的过程，逐步调整到最佳状态。 |
| -XX:ConcGCThreads=n                                          | 并发垃圾收集器使用的线程数量                                 | 默认值随JVM运行的平台不同而不同                              |
| -XX:G1MixedGCLiveThresholdPercent=65                         | 混合垃圾回收周期中要包括的旧区域设置<br/>占用率阈值          | 默认占用率为 65%                                             |
| -XX:G1MixedGCCountTarget=8                                   | 设置标记周期完成后，对存活数据上限为<br/>G1MixedGCLIveThresholdPercent 的旧<br/>区域执行混合垃圾回收的目标次数 | 默认8次混合垃圾回收，混合回收的目标是要控制在此目<br/>标次数以内 |
| -<br/>XX:G1OldCSetRegionThresholdPercent=1                   | 描述Mixed GC时，Old Region被加入到<br/>CSet中                | 默认情况下，G1只把10%的Old Region加入到CSet中                |

### 十一、 常用命令

#### 11.1 jps

> 查看java进程

#### 11.2  jinfo

> (1)实时查看和调整JVM配置参数

> (2)查看
> jinfo -flag name PID 查看某个java进程的name属性的值

```
jinfo -flag MaxHeapSize PID
jinfo -flag UseG1GC PID
```

> (3)修改
> 参数只有被标记为manageable的flags可以被实时修改
> jinfo -flag [+|-] PID
> jinfo -flag = PID

> (4)查看曾经赋过值的一些参数

```
jinfo -flags PID
```

#### 11.3  jstat

> (1)查看虚拟机性能统计信息

> (2)查看类装载信息

```
jstat -class PID 1000 10   查看某个java进程的类装载信息，每1000毫秒输出一次，共输出10
次
```

> (3)查看垃圾收集信息

```
jstat -gc PID 1000 10
```

#### 11.4  jstack

> (1)查看线程堆栈信息

> (2)用法

```
jstack PID
```

#### 11.5 jmap

> (1)生成堆转储快照

> (2)打印出堆内存相关信息

```
-XX:+PrintFlagsFinal -Xms300M -Xmx300M
jmap -heap PID
```

> (3)dump出堆内存相关信息
>
> jmap -dump:format=b,file=heap.hprof PID

```
jmap -dump:format=b,file=heap.hprof 44808
```

> (4)要是在发生堆内存溢出的时候，能自动dump出该文件就好了

一般在开发中，JVM参数可以加上下面两句，这样内存溢出时，会自动dump出该文件

-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=heap.hprof

### 十二、常用命令

#### 12.1 jconsole

JConsole工具是JDK自带的可视化监控工具。查看java应用程序的运行概况、监控堆信息、永久区使用
情况、类加载情况等。

> 命令行中输入：jconsole

#### 12.2  jvisualvm



#### 12.3 Arthas

> github ：https://github.com/alibaba/arthas
>
> Arthas 是Alibaba开源的Java诊断工具，采用命令行交互模式，是排查jvm相关问题的利器。
>
> ![image-20220718183936361](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220718183936361.png)

##### 12.3.1 下载安装

```shell
curl -O https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar
# 然后可以选择一个Java进程
```

**Print usage**

```
java -jar arthas-boot.jar -h
```

##### 12.3.2 常用命令

> 具体每个命令怎么使用，大家可以自己查阅资料

```
version:查看arthas版本号
help:查看命名帮助信息
cls:清空屏幕
session:查看当前会话信息
quit:退出arthas客户端
---
dashboard:当前进程的实时数据面板
thread:当前JVM的线程堆栈信息
jvm:查看当前JVM的信息
sysprop:查看JVM的系统属性
---
sc:查看JVM已经加载的类信息
dump:dump已经加载类的byte code到特定目录
jad:反编译指定已加载类的源码
---
monitor:方法执行监控
watch:方法执行数据观测
trace:方法内部调用路径，并输出方法路径上的每个节点上耗时
stack:输出当前方法被调用的调用路径
```

#### 12.4 GC日志分析工具

> 要想分析日志的信息，得先拿到GC日志文件才行，所以得先配置一下
> 根据前面参数的学习，下面的配置很容易看懂

```
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps
-Xloggc:gc.log
```

- 在线

  http://gceasy.io

- GCViewer