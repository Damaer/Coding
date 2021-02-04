[TOC]
最近在java的源代码中总是可以看到`isAssignableFrom()`这个方法，到底是干嘛的？怎么用？

# 1. isAssignableFrom()是干什么用的？
首先我们必须知道的是，java里面一切皆对象，类本身也是会当成对象来处理，主要体现在类的`.class`文件，其实加载到java虚拟机之后，也是一个对象，它就是`Class`对象,全限定类名:`java.lang.Class`。

那这个`isAssignableFrom()`其实就是Class对象的一个方法：
```java
    /**
     * Determines if the class or interface represented by this
     * {@code Class} object is either the same as, or is a superclass or
     * superinterface of, the class or interface represented by the specified
     * {@code Class} parameter. It returns {@code true} if so;
     * otherwise it returns {@code false}. If this {@code Class}
     * object represents a primitive type, this method returns
     * {@code true} if the specified {@code Class} parameter is
     * exactly this {@code Class} object; otherwise it returns
     * {@code false}.
     *
     * <p> Specifically, this method tests whether the type represented by the
     * specified {@code Class} parameter can be converted to the type
     * represented by this {@code Class} object via an identity conversion
     * or via a widening reference conversion. See <em>The Java Language
     * Specification</em>, sections 5.1.1 and 5.1.4 , for details.
     *
     * @param cls the {@code Class} object to be checked
     * @return the {@code boolean} value indicating whether objects of the
     * type {@code cls} can be assigned to objects of this class
     * @exception NullPointerException if the specified Class parameter is
     *            null.
     * @since JDK1.1
     */
    public native boolean isAssignableFrom(Class<?> cls);
```
用`native`关键字描述，说明是一个底层方法，实际上是使用c/c++实现的，java里面没有实现，那么这个方法是干什么的呢？我们从上面的注释可以解读：

如果是`A.isAssignableFrom(B)`
确定一个类(B)是不是继承来自于另一个父类(A)，一个接口(A)是不是实现了另外一个接口(B)，或者两个类相同。主要，这里比较的维度不是实例对象，而是类本身，因为这个方法本身就是`Class`类的方法，判断的肯定是和类信息相关的。

也就是判断当前的Class对象所表示的类，是不是参数中传递的Class对象所表示的类的父类，超接口，或者是相同的类型。是则返回true，否则返回false。
# 2.代码实验测试
## 2.1 父子继承关系测试
```java
class A{
}
class B extends A{
}
class C extends B{
}
public class test {
    public static void main(String[] args) {
        A a = new A();
        B b = new B();
        B b1 = new B();
        C c = new C();
        System.out.println(a.getClass().isAssignableFrom(a.getClass()));
        System.out.println(a.getClass().isAssignableFrom(b.getClass()));
        System.out.println(a.getClass().isAssignableFrom(c.getClass()));
        System.out.println(b1.getClass().isAssignableFrom(b.getClass()));

        System.out.println(b.getClass().isAssignableFrom(c.getClass()));

        System.out.println("=====================================");
        System.out.println(A.class.isAssignableFrom(a.getClass()));
        System.out.println(A.class.isAssignableFrom(b.getClass()));
        System.out.println(A.class.isAssignableFrom(c.getClass()));

        System.out.println("=====================================");
        System.out.println(Object.class.isAssignableFrom(a.getClass()));
        System.out.println(Object.class.isAssignableFrom(String.class));
        System.out.println(String.class.isAssignableFrom(Object.class));
    }
}
```
运行结果如下：
``` java
true
true
true
true
true
=====================================
true
true
true
=====================================
true
true
false
```

从运行结果来看，`C`继承于`B`，`B`继承于`A`,那么任何一个类都可以`isAssignableFrom`其本身，这个从中文意思来理解就是可以从哪一个装换而来，自身装换而来肯定是没有问题的，父类可以由子类装换而来也是没有问题的，所以A可以由B装换而来，同时也可以由子类的子类转换而来。

上面的代码也说明一点，所有的类，其最顶级的父类也是`Object`，也就是所有的类型都可以转换成为`Object`。

## 2.2 接口的实现关系测试
```java
interface InterfaceA{
}

class ClassB implements InterfaceA{

}
class ClassC implements InterfaceA{

}
class ClassD extends ClassB{

}
public class InterfaceTest {
    public static void main(String[] args) {
        System.out.println(InterfaceA.class.isAssignableFrom(InterfaceA.class));
        System.out.println(InterfaceA.class.isAssignableFrom(ClassB.class));
        System.out.println(InterfaceA.class.isAssignableFrom(ClassC.class));
        System.out.println(ClassB.class.isAssignableFrom(ClassC.class));
        System.out.println("============================================");

        System.out.println(ClassB.class.isAssignableFrom(ClassD.class));
        System.out.println(InterfaceA.class.isAssignableFrom(ClassD.class));
    }
}
```

输出结果如下：
```java
true
true
true
false
============================================
true
true
```

从上面的结果看，其实接口的实现关系和类的实现关系是一样的，没有什么区别，但是如果`B`和`C`都实现了同一个接口，他们之间其实是不能互转的。

如果`B`实现了接口`A`，`D`继承了`B`，实际上`D`是可以上转为A接口的，相当于`D`间接实现了`A`，这里也说明了一点，其实继承关系和接口实现关系，在`isAssignableFrom()`的时候是一样的，一视同仁的。


# 3.总结
`isAssignableFrom`是用来判断子类和父类的关系的，或者接口的实现类和接口的关系的，默认所有的类的终极父类都是`Object`。如果`A.isAssignableFrom(B)`结果是true，证明`B`可以转换成为`A`,也就是`A`可以由`B`转换而来。

这个方法在我们平时使用的不多，但是很多源码里面判断两个类之间的关系的时候，**（注意：是两个类的关系，不是两个类的实例对象的关系！！！）**，会使用到这个方法来判断，大概因为框架代码或者底层代码都是经过多层抽象，做到容易拓展和解耦合，只能如此。

**【作者简介】**：  
秦怀，公众号【**秦怀杂货店**】作者，技术之路不在一时，山高水长，纵使缓慢，驰而不息。这个世界希望一切都很快，更快，但是我希望自己能走好每一步，写好每一篇文章，期待和你们一起交流。

此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者核实删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~ 


![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201012000828.png)
