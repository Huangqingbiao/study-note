### Java数组

数组是存储在堆上的**对象**，可以保存多个**同类型**的变量

### Java枚举

枚举限制变量只能是预先设定好的值

### Java关键字

**assert**：断言条件是否满足

**final**:表示一个变量在初始化之后就不能再改变，被修饰的方法不能重写，被修饰的类不能有子类

**instanceof**：测试一个对象是否是某个类的实例

**native**:表示方法用非java代码实现

**synchronized**:表示同一时间只能用一个线程访问被修饰的代码块。

**this**:表示调用当前实例或调用另一个构造方法

**throw**:抛出异常

**throws**:定义方法可能抛出的异常（用在方法上）

**transient**:修饰不要序列化的字段

**volatile**:标记字段可以被多个线程同时访问，而不做同步

### Java类

**局部变量**：定义在方法，构造方法或语句块中的的变量被称为局部变量，变量的声明和初始化都是在方法中的，在方法结束后，变量会自动销毁。**局部变量是在栈上分配的**

**成员变量**：成员变量是定义在类中，方法体外的变量，成员变量是在**创建对象**的时候实例化的，成员变量可以被类中的方法、构造方法和类的语句块访问。

**类变量**：类变量也声明在类中，方法体外，但必须声明为static类型，它属于类的所有实例。

```
变量就是申请内存空间来存储值，也就是说，当创建变量的时候，需要在内存中申请空间，内存管理系统根据变量的类型为变量分配存储空间，分配的空间只能存储该类型的数据
```



### Java数据类型

**基本数据类型**

```
Java语言提供了8种基本数据类型。
int、long、short、byte、float、double、char、boolean
```

**引用数据类型**

```
引用数据类型由该类型的变量引用该类型的对象。
字符串，数组是引用数据类型
所有的引用数据类型的默认值是Null
```

### Java Number类

```
包装类（Integer、Long、Byte、Double、Float、Short)都是抽象类的子类
```

### Java String类

```
String类是不可改变的，一旦创建了String对象，那它的值就无法改变了。如果要对字符串做修改，可以选择StringBuffer类和StringBuilder类
```

### Java StringBuffer类和StringBuilder类

```
和String类不同得到是，StringBuffer和StringBuilder类的对象能够被多次的修改，并且不产生新的未使用对象

StringBuilder的方法不是线程安全的，所以它的速度相交与StringBuffer有优势。
StringBuffer的方法是线程安全的，速度较StringBuilder慢
```

**线程安全**

```
线程安全就是多线程访问时，采用了加锁机制，当一个线程访问该类的某个数据时，其他线程必须等到该线程读取完毕后才能访问，即对数据进行了保护，不会出现数据不一致或数据污染
```

Java语言中提供的数组是用来存储**固定大小**的**同类型**元素

### Java正则表达式

```
java.util.regex包下的类主要有一下三个：
Pattern类、Matcher类、PatternSyntaxException类
```

### Java流（Stream）文件（File)和IO

```
一个流可以理解为一个数据的序列，输入流表示从一个源读取数据，输出流表示向一个目标写数据
```

### Java重写（Override)与重载（Overload）

```
重写是子类对父类的允许访问的方法的实现过程进行重新编写，返回值和形参都不能改变。
```

### Java接口

```
除非实现接口的类是抽象类，否则该类必须定义接口中所有的方法
```

### Java数据结构

```
Java工具包提供了强大的数据结构，Java的数据结构主要包括以下几种接口和类
枚举（Enumeration）、位集合（BitSet）、向量（Vector）、栈（Stack）、字典（Dictionary）、哈希表（Hashtable）、属性（Properties）
```

### Java序列化

```
一个类的对象要想序列化成功，必须满足两个条件：
1、该类必须实现java.io.Serializable接口
2、该类的所有属性必须都是可序列化的，如果有一个属性不可序列化，则该属性必须注明是短暂的

检查一个类的实例是否能序列化，只需要检查该类有没有实现Java.io.Serializable接口
```

**ObjectOutputStream类**用来序列化一个对象

当序列化一个对象到文件时，按照Java标准约定是给文件一个.ser扩展名

```java
Employee e = new Employee(1,"小明");
//序列化对象
ObjectOutputStream out = new ObjectOutputStream("/tmp/employee.ser");
out.writeObject(e);
out.close();

//反序列化对象
ObjectInputStream in = new ObjectInputStream();
e = (Employee) in.readObject();
```

### Java多线程编程

```
进程：一个进程包括有操作系统分配的内存空间，包含一个或多个线程，一个线程不能独立存在，它必须是进程的一部分
```

**线程的生命周期**

```
新建状态(执行start()方法)--->就绪状态(系统调度，获取CPU资源，执行run()方法)--->运行状态（如果线程阻塞，让出CPU资源，回到就绪状态）--->死亡状态(run方法执行完成，可用stop与destory方法强制终止)
```

**线程的优先级**

```
MIN_PRIORITY(1)
MAX_PRIORITY(10)
NORM_PRIORITY(5)-----------默认
```

**创建线程**

```
Java提供三种创建线程的方式：
1、通过实现Runnable接口
2、通过继承Thread类
3、通过Callable和Future创建线程
```

### Java8新特性

**Lambda表达式**：Lamdba允许把函数作为一个方法的参数

### Java反射

**java.lang.Class类**

Class类对象表示**运行时程序**中的一个类











