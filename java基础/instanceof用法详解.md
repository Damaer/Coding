

# 1. instanceof关键字

如果你之前一直没有怎么仔细了解过`instanceof`关键字，现在就来了解一下：

<img src="https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201129214016.png" style="zoom:50%;" />

`instanceof`其实是java的一个二元操作符，和`=`,`<`,`>`这些是类似的，同时它也是被保留的关键字，主要的作用，是为了测试左边的对象，是不是右边的类的实例，返回的是boolean值。

```java
A instanceof B
```
注意：`A`是实例，而`B`则是`Class类`

下面使用代码测试一下：

```java
class A{
}
interface InterfaceA{

}
class B extends A implements InterfaceA{

}
public class Test {
    public static void main(String[] args) {
        B b = new B();
        System.out.println(b instanceof B);
        System.out.println(b instanceof A);
        System.out.println(b instanceof InterfaceA);
        
        A a = new A();
        System.out.println(a instanceof InterfaceA);
    }
}
```



输出结果如下：

```java
true
true
true
false
```

从上面的结果，其实我们可以看出`instanceof`，相当于判断当前**对象**能不能装换成为该类型，`java`里面上转型是安全的，子类对象可以转换成为父类对象，接口实现类对象可以装换成为接口对象。

对象`a`和`Interface`没有什么关系，所以返回`false`。



那如果我们装换成为Object了，它还能认出来是哪一个类的对象么？

```java
public class Test {
    public static void main(String[] args) {
        Object o = new ArrayList<Integer>();
        System.out.println(o instanceof ArrayList);

        String str = "hello world";
        System.out.println(str instanceof String);
        System.out.println(str instanceof Object);
    }
}
```

上面的结果返回都是`true`，也就是认出来还是哪一个类的对象。同时我们使用`String`对象测试的时候，发现对象既是`String`的实例，也是`Object`的实例，也印证了`Java`里面所有类型都默认继承了`Obejct`。



但是值得注意的是，我们只能使用对象来`instanceof`，不能使用基本数据类型，否则会报错。

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201129204933.png)



如果对象为`null`，那是什么类型？

这个答案是：不知道什么类型，因为`null`可以转换成为任何类型，所以不属于任何类型，`instanceof`结果会是`false`。

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201129205213.png)



具体的实现策略我们可以在官网找到：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201129205647.png)

如果`S`是`objectref`所引用的对象的类，而`T`是已解析类，数组或接口的类型，则`instanceof`确定是否 `objectref`是`T`的一个实例。`S s = new A(); s instanceof T`

- 如果S是一个普通的(非数组)类，则:
  - 如果T是一个类类型，那么S必须是T的同一个类，或者S必须是T的子类;
  - 如果T是接口类型，那么S必须实现接口T。

- 如果S是接口类型，则:
  - 如果T是类类型，那么T必须是Object。
  - 如果T是接口类型，那么T一定是与S相同的接口或S的超接口。

- 如果S是表示数组类型SC的类[]，即类型SC的组件数组，则:
  - 如果T是类类型，那么T必须是Object。
  - 如果T是一种接口类型，那么T必须是数组实现的接口之一(JLS§4.10.3)。
  - 如果T是一个类型为TC的数组[]，即一个类型为TC的组件数组，那么下列其中一个必须为真:
    - TC和SC是相同的原始类型。
    - TC和SC是引用类型，类型SC可以通过这些运行时规则转换为TC。





但是具体的底层原理我在知乎找到的**R大** 回答的相关问题，https://www.zhihu.com/question/21574535,看完觉得我太弱了...我是菜鸟...我确实是菜鸟



# 2. isInstance()方法

其实这个和上面那个是基本相同的，主要是这个调用者是`Class`对象，判断参数里面的对象是不是这个`Class`对象的实例。

```java
class A {
}

interface InterfaceA {

}

class B extends A implements InterfaceA {

}

public class Test {
    public static void main(String[] args) {
        B b = new B();
        System.out.println(B.class.isInstance(b));
        System.out.println(A.class.isInstance(b));
        System.out.println(InterfaceA.class.isInstance(b));

        A a = new A();
        System.out.println(InterfaceA.class.isInstance(a));
    }
}
```

历史总是惊人的相似！！！

```java
true
true
true
false
```

事实证明，这个`isInstance(o)`判断的是`o`是否属于当前`Class`类的实例.

不信？再来测试一下：

```java
public class Test {
    public static void main(String[] args) {
        String s = "hello";
        System.out.println(String.class.isInstance(s)); 				// true
        System.out.println(Object.class.isInstance(s)); 				// true

        
        System.out.println("=============================");
        Object o = new ArrayList<String>();
        System.out.println(String.class.isInstance(o));					// false
        System.out.println(ArrayList.class.isInstance(o));			// true
        System.out.println(Object.class.isInstance(o));					// true
    }
}
```

可以看出，其实就是装换成为`Object`，之前的类型信息还是会保留着，结果和`instance`一样，区别是：

- `instanceof` :前面是实例对象，后面是类型
- `isInstance`:调用者（前面）是类型对象，参数（后面）是实例对象



但是有一个区别哦😯，`isInstance()`这个方法，是可以使用在基本类型上的，其实也不算是用在基本类型，而是自动做了装箱操作。看下面👇：

```java
        System.out.println(Integer.class.isInstance(1));
```

参数里面的1，其实会被装箱成为`new Integer(1)`，所以这样用不会报错。



# 3. instanceof，isInstance，isAssignableFrom区别是什么？

- `instanceof` 判断对象和类型之间的关系，是关键字，只能用于对象实例，判断左边的对象是不是右边的类(包括父类)或者接口(包括父类)的实例化。
- `isInstance(Object o)`：判断对象和类型之间的关系，判断`o`是不是调用这个方法的`class`(包括父类)或者接口(包括父类)的实例化。
- `isAssignableFrom`:判断的是类和类之间的关系，调用者是否可以由参数中的`Class`对象转换而来。



注意：`java`里面一切皆是对象，所以，`class`本身也是对象。

**【作者简介】**：  
秦怀，公众号【**秦怀杂货店**】作者，技术之路不在一时，山高水长，纵使缓慢，驰而不息。这个世界希望一切都很快，更快，但是我希望自己能走好每一步，写好每一篇文章，期待和你们一起交流。

此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者核实删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~ 


![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201012000828.png)

