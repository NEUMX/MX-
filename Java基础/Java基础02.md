# Java基础02

# Java基础02
## 面向对象基础
### [](https://javaguide.cn/java/basis/#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E5%92%8C%E9%9D%A2%E5%90%91%E8%BF%87%E7%A8%8B%E7%9A%84%E5%8C%BA%E5%88%AB)面向对象和面向过程的区别
两者的主要区别在于解决问题的方式不同：

+ 面向过程把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题。
+ 面向对象会先抽象出对象，然后用对象执行方法的方式解决问题。

另外，面向对象开发的程序一般更易维护、易复用、易扩展。

相关 issue :[面向过程：面向过程性能比面向对象高？？](https://github.com/Snailclimb/JavaGuide/issues/431)[open in new window](https://github.com/Snailclimb/JavaGuide/issues/431)。

下面是一个求圆的面积和周长的示例，简单分别展示了面向对象和面向过程两种不同的解决方案。

**面向对象**：

```plain
public class Circle {
    // 定义圆的半径
    private double radius;

    // 构造函数
    public Circle(double radius) {
        this.radius = radius;
    }

    // 计算圆的面积
    public double getArea() {
        return Math.PI * radius * radius;
    }

    // 计算圆的周长
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }

    public static void main(String[] args) {
        // 创建一个半径为3的圆
        Circle circle = new Circle(3.0);

        // 输出圆的面积和周长
        System.out.println("圆的面积为：" + circle.getArea());
        System.out.println("圆的周长为：" + circle.getPerimeter());
    }
}
```

我们定义了一个Circle类来表示圆，该类包含了圆的半径属性和计算面积、周长的方法。

**面向过程**：

```java

public class Main {
    public static void main(String[] args) {
        // 定义圆的半径
        double radius = 3.0;

        // 计算圆的面积和周长
        double area = Math.PI * radius * radius;
        double perimeter = 2 * Math.PI * radius;

        // 输出圆的面积和周长
        System.out.println("圆的面积为：" + area);
        System.out.println("圆的周长为：" + perimeter);
    }
}
```

我们直接定义了圆的半径，并使用该半径直接计算出圆的面积和周长。

### [](https://javaguide.cn/java/basis/#%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%AF%B9%E8%B1%A1%E7%94%A8%E4%BB%80%E4%B9%88%E8%BF%90%E7%AE%97%E7%AC%A6-%E5%AF%B9%E8%B1%A1%E5%AE%9E%E4%BD%93%E4%B8%8E%E5%AF%B9%E8%B1%A1%E5%BC%95%E7%94%A8%E6%9C%89%E4%BD%95%E4%B8%8D%E5%90%8C)创建一个对象用什么运算符?对象实体与对象引用有何不同?
new 运算符，new 创建对象实例（对象实例在

内存中），对象引用指向对象实例（对象引用存放在栈内存中）。

+ 一个对象引用可以指向 0 个或 1 个对象（一根绳子可以不系气球，也可以系一个气球）；
+ 一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）。

### [](https://javaguide.cn/java/basis/#%E5%AF%B9%E8%B1%A1%E7%9A%84%E7%9B%B8%E7%AD%89%E5%92%8C%E5%BC%95%E7%94%A8%E7%9B%B8%E7%AD%89%E7%9A%84%E5%8C%BA%E5%88%AB)对象的相等和引用相等的区别
+ 对象的相等一般比较的是内存中存放的内容是否相等。
+ 引用相等一般比较的是他们指向的内存地址是否相等。

这里举一个例子：

```plain
String str1 = "hello";
String str2 = new String("hello");
String str3 = "hello";
// 使用 == 比较字符串的引用相等
System.out.println(str1 == str2);
System.out.println(str1 == str3);
// 使用 equals 方法比较字符串的相等
System.out.println(str1.equals(str2));
System.out.println(str1.equals(str3));
```

输出结果：

```plain
false
true
true
true
```

从上面的代码输出结果可以看出：

+ str1和str2不相等，而str1和str3相等。这是因为==运算符比较的是字符串的引用是否相等。
+ str1、str2、str3三者的内容都相等。这是因为equals方法比较的是字符串的内容，即使这些字符串的对象引用不同，只要它们的内容相等，就认为它们是相等的。

### [](https://javaguide.cn/java/basis/#%E5%A6%82%E6%9E%9C%E4%B8%80%E4%B8%AA%E7%B1%BB%E6%B2%A1%E6%9C%89%E5%A3%B0%E6%98%8E%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95-%E8%AF%A5%E7%A8%8B%E5%BA%8F%E8%83%BD%E6%AD%A3%E7%A1%AE%E6%89%A7%E8%A1%8C%E5%90%97)如果一个类没有声明构造方法，该程序能正确执行吗?
构造方法是一种特殊的方法，主要作用是完成对象的初始化工作。

如果一个类没有声明构造方法，也可以执行！因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。如果我们自己添加了类的构造方法（无论是否有参），Java 就不会添加默认的无参数的构造方法了。

我们一直在不知不觉地使用构造方法，这也是为什么我们在创建对象的时候后面要加一个括号（因为要调用无参的构造方法）。如果我们重载了有参的构造方法，记得都要把无参的构造方法也写出来（无论是否用到），因为这可以帮助我们在创建对象的时候少踩坑。

### [](https://javaguide.cn/java/basis/#%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B%E7%89%B9%E7%82%B9-%E6%98%AF%E5%90%A6%E5%8F%AF%E8%A2%AB-override)构造方法有哪些特点？是否可被 override?
构造方法特点如下：

+ 名字与类名相同。
+ 没有返回值，但不能用 void 声明构造函数。
+ 生成类的对象时自动执行，无需调用。

构造方法不能被 override（重写）,但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况。

### [](https://javaguide.cn/java/basis/#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%89%E5%A4%A7%E7%89%B9%E5%BE%81)面向对象三大特征
#### [](https://javaguide.cn/java/basis/#%E5%B0%81%E8%A3%85)封装
封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。就好像我们看不到挂在墙上的空调的内部的零件信息（也就是属性），但是可以通过遥控器（方法）来控制空调。如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。就好像如果没有空调遥控器，那么我们就无法操控空凋制冷，空调本身就没有意义了（当然现在还有很多其他方法 ，这里只是为了举例子）。

```plain
public class Student {
    private int id;//id属性私有化
    private String name;//name属性私有化

    //获取id的方法
    public int getId() {
        return id;
    }

    //设置id的方法
    public void setId(int id) {
        this.id = id;
    }

    //获取name的方法
    public String getName() {
        return name;
    }

    //设置name的方法
    public void setName(String name) {
        this.name = name;
    }
}
```

#### [](https://javaguide.cn/java/basis/#%E7%BB%A7%E6%89%BF)继承
不同类型的对象，相互之间经常有一定数量的共同点。例如，小明同学、小红同学、小李同学，都共享学生的特性（班级、学号等）。同时，每一个对象还定义了额外的特性使得他们与众不同。例如小明的数学比较好，小红的性格惹人喜爱；小李的力气比较大。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

**关于继承如下 3 点请记住：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。

#### [](https://javaguide.cn/java/basis/#%E5%A4%9A%E6%80%81)多态
多态，顾名思义，表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

**多态的特点:**

+ 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；
+ 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；
+ 多态不能调用“只在子类存在但在父类不存在”的方法；
+ 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。

### [](https://javaguide.cn/java/basis/#%E6%8E%A5%E5%8F%A3%E5%92%8C%E6%8A%BD%E8%B1%A1%E7%B1%BB%E6%9C%89%E4%BB%80%E4%B9%88%E5%85%B1%E5%90%8C%E7%82%B9%E5%92%8C%E5%8C%BA%E5%88%AB)接口和抽象类有什么共同点和区别？
**共同点**：

+ 都不能被实例化。
+ 都可以包含抽象方法。
+ 都可以有默认实现的方法（Java 8 可以用default关键字在接口中定义默认方法）。

**区别**：

+ 接口主要用于对类的行为进行约束，你实现了某个接口就具有了对应的行为。抽象类主要用于代码复用，强调的是所属关系。
+ 一个类只能继承一个类，但是可以实现多个接口。
+ 接口中的成员变量只能是public static final类型的，不能被修改且必须有初始值，而抽象类的成员变量默认 default，可在子类中被重新定义，也可被重新赋值。

### [](https://javaguide.cn/java/basis/#%E6%B7%B1%E6%8B%B7%E8%B4%9D%E5%92%8C%E6%B5%85%E6%8B%B7%E8%B4%9D%E5%8C%BA%E5%88%AB%E4%BA%86%E8%A7%A3%E5%90%97-%E4%BB%80%E4%B9%88%E6%98%AF%E5%BC%95%E7%94%A8%E6%8B%B7%E8%B4%9D)深拷贝和浅拷贝区别了解吗？什么是引用拷贝？
关于深拷贝和浅拷贝区别，我这里先给结论：

+ **浅拷贝**：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址，也就是说拷贝对象和原对象共用同一个内部对象。
+ **深拷贝**：深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

上面的结论没有完全理解的话也没关系，我们来看一个具体的案例！

#### [](https://javaguide.cn/java/basis/#%E6%B5%85%E6%8B%B7%E8%B4%9D)浅拷贝
浅拷贝的示例代码如下，我们这里实现了Cloneable接口，并重写了clone()方法。

clone()方法的实现很简单，直接调用的是父类Object的clone()方法。

```plain
public class Address implements Cloneable{
    private String name;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Person implements Cloneable {
    private Address address;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

测试：

```plain
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// true
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出，person1的克隆对象和person1使用的仍然是同一个Address对象。

#### [](https://javaguide.cn/java/basis/#%E6%B7%B1%E6%8B%B7%E8%B4%9D)深拷贝
这里我们简单对Person类的clone()方法进行修改，连带着要把Person对象内部的Address对象一起复制。

```plain
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

测试：

```plain
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// false
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出，显然person1的克隆对象和person1包含的Address对象已经是不同的了。

**那什么是引用拷贝呢？**简单来说，引用拷贝就是两个不同的引用指向同一个对象。

我专门画了一张图来描述浅拷贝、深拷贝、引用拷贝：

![1732497551484-92dd4393-f4b0-457e-a39e-1d09c87a45e7.png](./img/jytZhzKuaqQL-e0i/1732497551484-92dd4393-f4b0-457e-a39e-1d09c87a45e7-964753.png)

浅拷贝、深拷贝、引用拷贝示意图

## [](https://javaguide.cn/java/basis/#object)Object
### [](https://javaguide.cn/java/basis/#object-%E7%B1%BB%E7%9A%84%E5%B8%B8%E8%A7%81%E6%96%B9%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)Object 类的常见方法有哪些？
Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法：

```plain
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * native 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以纳秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 纳秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { }
```

### [](https://javaguide.cn/java/basis/#%E5%92%8C-equals-%E7%9A%84%E5%8C%BA%E5%88%AB)== 和 equals() 的区别
**==**对于基本类型和引用类型的作用效果是不同的：

+ 对于基本数据类型来说，==比较的是值。
+ 对于引用数据类型来说，==比较的是对象的内存地址。

因为 Java 只有值传递，所以，对于 == 来说，不管是比较基本数据类型，还是引用数据类型的变量，其本质比较的都是值，只是引用类型变量存的值是对象的地址。

**equals()**不能用于判断基本数据类型的变量，只能用来判断两个对象是否相等。equals()方法存在于Object类中，而Object类是所有类的直接或间接父类，因此所有的类都有equals()方法。

Object类equals()方法：

```plain
public boolean equals(Object obj) {
     return (this == obj);
}
```

equals()方法存在两种使用情况：

+ *_类没有重写_***equals()****方法**：通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象，使用的默认是Object类equals()方法。
+ *_类重写了_***equals()****方法**：一般我们都重写equals()方法来比较两个对象中的属性是否相等；若它们的属性相等，则返回 true(即，认为这两个对象相等)。

举个例子（这里只是为了举例。实际上，你按照下面这种写法的话，像 IDEA 这种比较智能的 IDE 都会提示你将==换成equals()）：

```plain
String a = new String("ab"); // a 为一个引用
String b = new String("ab"); // b为另一个引用,对象的内容一样
String aa = "ab"; // 放在常量池中
String bb = "ab"; // 从常量池中查找
System.out.println(aa == bb);// true
System.out.println(a == b);// false
System.out.println(a.equals(b));// true
System.out.println(42 == 42.0);// true
```

String中的equals方法是被重写过的，因为Object的equals方法是比较的对象的内存地址，而String的equals方法比较的是对象的值。

当创建String类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个String对象。

String类equals()方法：

```plain
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

### [](https://javaguide.cn/java/basis/#hashcode-%E6%9C%89%E4%BB%80%E4%B9%88%E7%94%A8)hashCode() 有什么用？
hashCode()的作用是获取哈希码（int整数），也称为散列码。这个哈希码的作用是确定该对象在哈希表中的索引位置。

![1732497551708-7a70ea11-2e44-4b13-ac5b-1fff493fac66.png](./img/jytZhzKuaqQL-e0i/1732497551708-7a70ea11-2e44-4b13-ac5b-1fff493fac66-329356.png)

hashCode() 方法

hashCode()定义在 JDK 的Object类中，这就意味着 Java 中的任何类都包含有hashCode()函数。另外需要注意的是：Object的hashCode()方法是本地方法，也就是用 C 语言或 C++ 实现的。

⚠️ 注意：该方法在**Oracle OpenJDK8**中默认是 "使用线程局部状态来实现 Marsaglia's xor-shift 随机数生成", 并不是 "地址" 或者 "地址转换而来", 不同 JDK/VM 可能不同在**Oracle OpenJDK8**中有六种生成方式 (其中第五种是返回地址), 通过添加 VM 参数: -XX:hashCode=4 启用第五种。参考源码:

+ [https://hg.openjdk.org/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/globals.hpp（1127](https://hg.openjdk.org/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/globals.hpp%EF%BC%881127)[open in new window](https://hg.openjdk.org/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/globals.hpp%EF%BC%881127)行）
+ [https://hg.openjdk.org/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp（537](https://hg.openjdk.org/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp%EF%BC%88537)[open in new window](https://hg.openjdk.org/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp%EF%BC%88537)行开始）

```plain
public native int hashCode();
```

散列表存储的是键值对(key-value)，它的特点是：**能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）**

### [](https://javaguide.cn/java/basis/#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9C%89-hashcode)为什么要有 hashCode？
我们以“HashSet如何检查重复”为例子来说明为什么要有hashCode？

下面这段内容摘自我的 Java 启蒙书《Head First Java》:

当你把对象加入HashSet时，HashSet会先计算对象的hashCode值来判断对象加入的位置，同时也会与其他已经加入的对象的hashCode值作比较，如果没有相符的hashCode，HashSet会假设对象没有重复出现。但是如果发现有相同hashCode值的对象，这时会调用equals()方法来检查hashCode相等的对象是否真的相同。如果两者相同，HashSet就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了equals的次数，相应就大大提高了执行速度。

其实，hashCode()和equals()都是用于比较两个对象是否相等。

**那为什么 JDK 还要同时提供这两个方法呢？**

这是因为在一些容器（比如HashMap、HashSet）中，有了hashCode()之后，判断元素是否在对应容器中的效率会更高（参考添加元素进HashSet的过程）！

我们在前面也提到了添加元素进HashSet的过程，如果HashSet在对比的时候，同样的hashCode有多个对象，它会继续使用equals()来判断是否真的相同。也就是说hashCode帮助我们大大缩小了查找成本。

*_那为什么不只提供_***hashCode()****方法呢？**

这是因为两个对象的hashCode值相等并不代表两个对象就相等。

**那为什么两个对象有相同的****hashCode****值，它们也不一定是相等的？**

因为hashCode()所使用的哈希算法也许刚好会让多个对象传回相同的哈希值。越糟糕的哈希算法越容易碰撞，但这也与数据值域分布的特性有关（所谓哈希碰撞也就是指的是不同的对象得到相同的hashCode)。

总结下来就是：

+ 如果两个对象的hashCode值相等，那这两个对象不一定相等（哈希碰撞）。
+ 如果两个对象的hashCode值相等并且equals()方法也返回true，我们才认为这两个对象相等。
+ 如果两个对象的hashCode值不相等，我们就可以直接认为这两个对象不相等。

相信大家看了我前面对hashCode()和equals()的介绍之后，下面这个问题已经难不倒你们了。

### [](https://javaguide.cn/java/basis/#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%87%8D%E5%86%99-equals-%E6%97%B6%E5%BF%85%E9%A1%BB%E9%87%8D%E5%86%99-hashcode-%E6%96%B9%E6%B3%95)为什么重写 equals() 时必须重写 hashCode() 方法？
因为两个相等的对象的hashCode值必须是相等。也就是说如果equals方法判断两个对象是相等的，那这两个对象的hashCode值也要相等。

如果重写equals()时没有重写hashCode()方法的话就可能会导致equals方法判断是相等的两个对象，hashCode值却不相等。

**思考**：重写equals()时没有重写hashCode()方法的话，使用HashMap可能会出现什么问题。

**总结**：

+ equals方法判断两个对象是相等的，那这两个对象的hashCode值也要相等。
+ 两个对象有相同的hashCode值，他们也不一定是相等的（哈希碰撞）。

更多关于hashCode()和equals()的内容可以查看：[Java hashCode() 和 equals()的若干问题解答](https://www.cnblogs.com/skywang12345/p/3324958.html)[open in new window](https://www.cnblogs.com/skywang12345/p/3324958.html)

## [](https://javaguide.cn/java/basis/#string)String
### [](https://javaguide.cn/java/basis/#string%E3%80%81stringbuffer%E3%80%81stringbuilder-%E7%9A%84%E5%8C%BA%E5%88%AB)String、StringBuffer、StringBuilder 的区别？
**可变性**

String是不可变的（后面会详细分析原因）。

StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串，不过没有使用final和private关键字修饰，最关键的是这个AbstractStringBuilder类还提供了很多修改字符串的方法比如append方法。

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
    //...
}
```

**线程安全性**

String中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder是StringBuilder与StringBuffer的公共父类，定义了一些字符串的基本操作，如expandCapacity、append、insert、indexOf等公共方法。StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。

**性能**

每次对String类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String对象。StringBuffer每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用StringBuilder相比使用StringBuffer仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据: 适用String
2. 单线程操作字符串缓冲区下操作大量数据: 适用StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据: 适用StringBuffer

### [](https://javaguide.cn/java/basis/#string-%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF%E4%B8%8D%E5%8F%AF%E5%8F%98%E7%9A%84)String 为什么是不可变的?
String类中使用final关键字修饰字符数组来保存字符串，~~所以~~~~String~~~~对象是不可变的。~~

```plain
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
    //...
}
```

🐛 修正：我们知道被final关键字修饰的类不能被继承，修饰的方法不能被重写，修饰的变量是基本数据类型则值不能改变，修饰的变量是引用类型则不能再指向其他对象。因此，final关键字修饰的数组保存字符串并不是String不可变的根本原因，因为这个数组保存的字符串是可变的（final修饰引用类型变量的情况）。String真正不可变有下面几点原因：

1. 保存字符串的数组被final修饰且为私有的，并且String类没有提供/暴露修改这个字符串的方法。
2. String类被final修饰导致其不能被继承，进而避免了子类破坏String不可变。

相关阅读：[如何理解 String 类型值的不可变？ - 知乎提问](https://www.zhihu.com/question/20618891/answer/114125846)[open in new window](https://www.zhihu.com/question/20618891/answer/114125846)补充（来自[issue 675](https://github.com/Snailclimb/JavaGuide/issues/675)[open in new window](https://github.com/Snailclimb/JavaGuide/issues/675)）：在 Java 9 之后，String、StringBuilder与StringBuffer的实现改用byte数组存储字符串。

```plain
public final class String implements java.io.Serializable,Comparable<String>, CharSequence {
    // @Stable 注解表示变量最多被修改一次，称为“稳定的”。
    @Stable
    private final byte[] value;
}

abstract class AbstractStringBuilder implements Appendable, CharSequence {
    byte[] value;

}
```

*_Java 9 为何要将__**String**__的底层实现由__*__char[]__**改成了**__byte[]__*_*?**新版的 String 其实支持两个编码方案：Latin-1 和 UTF-16。如果字符串中包含的汉字没有超过 Latin-1 可表示范围内的字符，那就会使用 Latin-1 作为编码方案。Latin-1 编码方案下，byte占一个字节(8 位)，char占用 2 个字节（16），byte相较char节省一半的内存空间。JDK 官方就说了绝大部分字符串对象只包含 Latin-1 可表示的字符。

![1732497551794-837ef119-613e-4671-abd6-a5bbb8f14e0b.png](./img/jytZhzKuaqQL-e0i/1732497551794-837ef119-613e-4671-abd6-a5bbb8f14e0b-329050.png)

如果字符串中包含的汉字超过 Latin-1 可表示范围内的字符，byte和char所占用的空间是一样的。这是官方的介绍：[https://openjdk.java.net/jeps/254](https://openjdk.java.net/jeps/254)[open in new window](https://openjdk.java.net/jeps/254)。

### [](https://javaguide.cn/java/basis/#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%8B%BC%E6%8E%A5%E7%94%A8-%E8%BF%98%E6%98%AF-stringbuilder)字符串拼接用“+” 还是 StringBuilder?
Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

```plain
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

上面的代码对应的字节码如下：

![1732497551909-9ce9440e-05e1-4590-8c41-400bb734386b.png](./img/jytZhzKuaqQL-e0i/1732497551909-9ce9440e-05e1-4590-8c41-400bb734386b-446033.png)

可以看出，字符串对象通过“+”的字符串拼接方式，实际上是通过StringBuilder调用append()方法实现的，拼接完成之后调用toString()得到一个String对象 。

不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个****StringBuilder****以复用，会导致创建过多的****StringBuilder****对象**。

```plain
String[] arr = {"he", "llo", "world"};
String s = "";
for (int i = 0; i < arr.length; i++) {
    s += arr[i];
}
System.out.println(s);
```

StringBuilder对象是在循环内部被创建的，这意味着每循环一次就会创建一个StringBuilder对象。

![1732497552048-d20e876e-f7fe-4c50-8b27-0dad6499abd7.png](./img/jytZhzKuaqQL-e0i/1732497552048-d20e876e-f7fe-4c50-8b27-0dad6499abd7-134415.png)

如果直接使用StringBuilder对象进行字符串拼接的话，就不会存在这个问题了。

```plain
String[] arr = {"he", "llo", "world"};
StringBuilder s = new StringBuilder();
for (String value : arr) {
    s.append(value);
}
System.out.println(s);
```

![1732497552121-1723239a-35f6-48b9-b9a8-22c020d0e266.png](./img/jytZhzKuaqQL-e0i/1732497552121-1723239a-35f6-48b9-b9a8-22c020d0e266-641441.png)

如果你使用 IDEA 的话，IDEA 自带的代码检查机制也会提示你修改代码。

不过，使用 “+” 进行字符串拼接会产生大量的临时对象的问题在 JDK9 中得到了解决。在 JDK9 当中，字符串相加 “+” 改为了用动态方法makeConcatWithConstants()来实现，而不是大量的StringBuilder了。这个改进是 JDK9 的[JEP 280](https://openjdk.org/jeps/280)[open in new window](https://openjdk.org/jeps/280)提出的，这也意味着 JDK 9 之后，你可以放心使用“+” 进行字符串拼接了。关于这部分改进的详细介绍，推荐阅读这篇文章：还在无脑用[StringBuilder？来重温一下字符串拼接吧](https://juejin.cn/post/7182872058743750715)[open in new window](https://juejin.cn/post/7182872058743750715)。

### [](https://javaguide.cn/java/basis/#string-equals-%E5%92%8C-object-equals-%E6%9C%89%E4%BD%95%E5%8C%BA%E5%88%AB)Stringequals() 和 Objectequals() 有何区别？
String中的equals方法是被重写过的，比较的是 String 字符串的值是否相等。Object的equals方法是比较的对象的内存地址。

### [](https://javaguide.cn/java/basis/#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%B8%B8%E9%87%8F%E6%B1%A0%E7%9A%84%E4%BD%9C%E7%94%A8%E4%BA%86%E8%A7%A3%E5%90%97)字符串常量池的作用了解吗？
**字符串常量池**是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

```plain
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String aa = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String bb = "ab";
System.out.println(aa==bb);// true
```

更多关于字符串常量池的介绍可以看一下[Java 内存区域详解](https://javaguide.cn/java/jvm/memory-area.html)[open in new window](https://javaguide.cn/java/jvm/memory-area.html)这篇文章。

### [](https://javaguide.cn/java/basis/#string-s1-new-string-abc-%E8%BF%99%E5%8F%A5%E8%AF%9D%E5%88%9B%E5%BB%BA%E4%BA%86%E5%87%A0%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%AF%B9%E8%B1%A1)String s1 = new String("abc");这句话创建了几个字符串对象？
会创建 1 或 2 个字符串对象。

1、如果字符串常量池中不存在字符串对象“abc”的引用，那么它会在堆上创建两个字符串对象，其中一个字符串对象的引用会被保存在字符串常量池中。

示例代码（JDK 1.8）：

```plain
String s1 = new String("abc");
```

对应的字节码：

![](https://cdn.nlark.com/yuque/0/2024/png/45178513/1732497552194-be3b023d-e51b-447d-8ea6-45d58061bf1c.png)

ldc命令用于判断字符串常量池中是否保存了对应的字符串对象的引用，如果保存了的话直接返回，如果没有保存的话，会在堆中创建对应的字符串对象并将该字符串对象的引用保存到字符串常量池中。

2、如果字符串常量池中已存在字符串对象“abc”的引用，则只会在堆中创建 1 个字符串对象“abc”。

示例代码（JDK 1.8）：

```plain
// 字符串常量池中已存在字符串对象“abc”的引用
String s1 = "abc";
// 下面这段代码只会在堆中创建 1 个字符串对象“abc”
String s2 = new String("abc");
```

对应的字节码：

![1732497552273-7b6ee564-326e-4614-9fa0-5c291335d3f4.png](./img/jytZhzKuaqQL-e0i/1732497552273-7b6ee564-326e-4614-9fa0-5c291335d3f4-013563.png)

这里就不对上面的字节码进行详细注释了，7 这个位置的ldc命令不会在堆中创建新的字符串对象“abc”，这是因为 0 这个位置已经执行了一次ldc命令，已经在堆中创建过一次字符串对象“abc”了。7 这个位置执行ldc命令会直接返回字符串常量池中字符串对象“abc”对应的引用。

### [](https://javaguide.cn/java/basis/#string-intern-%E6%96%B9%E6%B3%95%E6%9C%89%E4%BB%80%E4%B9%88%E4%BD%9C%E7%94%A8)Stringintern 方法有什么作用?
String.intern()是一个 native（本地）方法，其作用是将指定的字符串对象的引用保存在字符串常量池中，可以简单分为两种情况：

+ 如果字符串常量池中保存了对应的字符串对象的引用，就直接返回该引用。
+ 如果字符串常量池中没有保存了对应的字符串对象的引用，那就在常量池中创建一个指向该字符串对象的引用并返回。

示例代码（JDK 1.8） :

```plain
// 在堆中创建字符串对象”Java“
// 将字符串对象”Java“的引用保存在字符串常量池中
String s1 = "Java";
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s2 = s1.intern();
// 会在堆中在单独创建一个字符串对象
String s3 = new String("Java");
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s4 = s3.intern();
// s1 和 s2 指向的是堆中的同一个对象
System.out.println(s1 == s2); // true
// s3 和 s4 指向的是堆中不同的对象
System.out.println(s3 == s4); // false
// s1 和 s4 指向的是堆中的同一个对象
System.out.println(s1 == s4); //true
```

### [](https://javaguide.cn/java/basis/#string-%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%8F%98%E9%87%8F%E5%92%8C%E5%B8%B8%E9%87%8F%E5%81%9A-%E8%BF%90%E7%AE%97%E6%97%B6%E5%8F%91%E7%94%9F%E4%BA%86%E4%BB%80%E4%B9%88)String 类型的变量和常量做“+”运算时发生了什么？
先来看字符串不加final关键字拼接的情况（JDK1.8）：

```plain
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";
String str4 = str1 + str2;
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

**注意**：比较 String 字符串的值是否相等，可以使用equals()方法。String中的equals方法是被重写过的。Object的equals方法是比较的对象的内存地址，而String的equals方法比较的是字符串的值是否相等。如果你使用==比较两个字符串是否相等的话，IDEA 还是提示你使用equals()方法替换。

![1732497552368-ed1f5964-591d-4244-b0a4-e76bc876a4e4.png](./img/jytZhzKuaqQL-e0i/1732497552368-ed1f5964-591d-4244-b0a4-e76bc876a4e4-623875.png)

**对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。**

在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做**常量折叠(Constant Folding)**的代码优化。《深入理解 Java 虚拟机》中是也有介绍到：

![1732497552474-bf6f51c4-5eeb-4e3e-87d9-224994cc5c1b.png](./img/jytZhzKuaqQL-e0i/1732497552474-bf6f51c4-5eeb-4e3e-87d9-224994cc5c1b-552958.png)

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于String str3 = "str" + "ing";编译器会给你优化成String str3 = "string";。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

+ 基本数据类型(byte、boolean、short、char、int、float、long、double)以及字符串常量。
+ final修饰的基本数据类型和字符串变量
+ 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过StringBuilder调用append()方法实现的，拼接完成之后调用toString()得到一个String对象 。

```plain
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用StringBuilder或者StringBuffer。

不过，字符串使用final关键字声明之后，可以让编译器当做常量来处理。

示例代码：

```plain
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
```

被final关键字修饰之后的String会被编译器当做常量来处理，编译器在程序编译期就可以确定它的值，其效果就相当于访问常量。

如果 ，编译器在运行时才能知道其确切值的话，就无法对其优化。

示例代码（str2在运行时才能确定其值）：

```plain
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的对象
System.out.println(c == d);// false
public static String getStr() {
      return "ing";
}
```

## [](https://javaguide.cn/java/basis/#%E5%8F%82%E8%80%83)参考
+ 深入解析 Stringintern：[https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)[open in new window](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)
+ R 大（RednaxelaFX）关于常量折叠的回答：[https://www.zhihu.com/question/55976094/answer/147302764](https://www.zhihu.com/question/55976094/answer/147302764)



> 更新: 2024-01-02 19:51:34  
原文: [https://www.yuque.com/vip6688/neho4x/cqxgd26sp40gvtaq](https://www.yuque.com/vip6688/neho4x/cqxgd26sp40gvtaq)
>



> 更新: 2024-11-25 10:11:24  
> 原文: <https://www.yuque.com/neumx/laxg2e/032f3f24501073719bf636f51fbb4a94>