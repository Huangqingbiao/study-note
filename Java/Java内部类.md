## Java内部类

​	内部类（内部定义的普通类、抽象类、接口的统称）是指一种嵌套的结构关系，即在一个类的内部定义一个类结构。内部类可以方便地访问外部类的私有属性，外部类也同样可以访问内部类的私有属性，内部类通常为外部类提供服务而存在。

​	内部类虽然定义在外部类的内部，但本身也是一个完整的类，也可以进行实例化，但实例化内部类必须先获取相应的外部类实例对象后，才可以利用外部类的实例化对象进行内部类对象的实例化操作

```java
外部类.内部类 内部类对象 = new 外部类().内部类();
```

> 内部类私有化

​	如果一个内部类不希望别其他类所使用，可以使用private关键字修饰符将这个内部化定义为私有内部类

```java
//	private、protected定义类的结构只允许使用在内部类声明中
class Outer{
  private class Inner{
  }
}
```



> 静态内部类

​	使用static关键字定义的内部类不再受到外部类实例化对象的影响，等同于一个“外部类”，名称为“外部类.内部类”，使用static关键字定义的内部类只能够调用外部类中的static定义的结构，并且在进行内部类实例化的时候也不再需要先获取外部类实例化对象，静态内部类对象实例化格式为:

```java
外部类.内部类 对象名称 = new 外部类.内部类();
```



> 方法内部类

​	内部类理论上可以在类的任意位置上进行定义，包括在代码块和普通方法中。

```java
//	在JDK1.8之前，如果方法中定义的内部类想要访问参数和局部变量，那么就需要使用final关键字修饰，而在JDK1.8后则不再需要。
class Out{
  public void fun(fianl long time){
    class Inner{
      
    }
  }
}
```



> 匿名内部类

​	接口或抽象类定义后都需要有一个子类来实现接口或抽象类中的抽象方法，但是在很多时候某些子类可能只使用一次，那么单独为接口或抽象类创建一个子类就显得浪费类，此时可以使用匿名内部类来解决此类问题

```java
interface Imessage{
  public void send(String message);
}
Imessage msg = new Imessage(){
  public void send(String message){
    
  }
}

interface Imessage{
  public void send(String str);
  public static Imessage getInstance(){
    return new Imessage(){
      public
    }
  }
}
```

由于匿名内部没有具体的名称，所有只能使用一次，匿名内部类多数情况下都是结合接口或抽象类使用