
> * 在java中，通常初学者搞不懂接口与抽象类，这也是面试比较容易问到的一个问题。下面我来谈谈自己的理解。如有不妥之处，还望批评指正，不胜感激。

[TOC]
## 1.抽象类怎么定义和继承？
我们定义一个抽象类`person.class`表示类（人）:
```java
//使用关键字abstract
public abstract class person {
    //吃东西的抽象方法,已经有所实现
    public void eat(){
        System.out.println("我是抽象方法吃东西");
    }
    
    //public 修饰的空实现的方法
    public void run(){}
    
    //无修饰，空实现
    void walk(){}
    
    //protected修饰的方法，空实现
    protected void sleep(){}
    
    //private修饰的空实现方法
    private void read(){}
}

```

- 1.抽象类使用abstract修饰，可以有抽象方法，也可以完全没有抽象方法，也可以是实现了的方法，但是所有的方法必须实现，空实现(`public void walk(){}`)也是实现的一种，而不能写 ~~`public void eat()`~~,后面必须带大括号。
- 2.方法修饰符可以使`public`,`protected`,`private`，或者是没有，没有默认为只能在同一个包下面继承,如果是`private`那么子类继承的时候就无法继承这个方法，也没有办法进行修改.
- 下面我们来写一个`Teacher.class`继承抽象类


同一个包下继承：
![](https://img-blog.csdnimg.cn/img_convert/bd3199b57749b6a290d6c21fd8a38da3.png)

不同的包下面继承：
![](https://img-blog.csdnimg.cn/img_convert/3791b40aff8c13be5ee211d029660513.png)

同个包下正确的代码如下(不重写私有的方法)：
```java
public class teacher extends person {
    @Override
    public void run(){
        System.out.println("我是实体类的方法跑步");
    }
    @Override
    void walk(){
        System.out.println("我是实体类的方法走路");
    }
    @Override
    protected void sleep(){
        System.out.println("我是实体类的方法睡觉");
    }
}

```
- 结果如下(**没有覆盖抽象类吃东西的方法，所以会调用抽象类默认的**):

![](https://img-blog.csdnimg.cn/img_convert/5a34eda5ca7e887ef5a64ce42658f9e5.png)

- 下面代码是重写了`eat()`方法的代码，重写是即使没有使用`@Override`也是起作用的：
```java
public class teacher extends person {
    public void eat(){
        System.out.println("我是实体类的方法吃东西");
    }
    @Override
    public void run(){
        System.out.println("我是实体类的方法跑步");
    }
    @Override
    void walk(){
        System.out.println("我是实体类的方法走路");
    }
    @Override
    protected void sleep(){
        System.out.println("我是实体类的方法睡觉");
    }
}
```
- 结果如下，吃东西的方法被覆盖掉了：
![](https://img-blog.csdnimg.cn/img_convert/ab654ad52453bf6a1e4a33d3dcc38f9b.png)

- 抽象类不能被实例化，比如：
![](https://img-blog.csdnimg.cn/img_convert/f190470d45ceb45fa2941da283ddeb6f.png)

- 子类可以实现抽象类的方法，也可以不实现，也可以只实现一部分，这样跑起来都是没有问题的，不实现的话，调用是默认使用抽象类的空实现，也就是什么都没有输出，要是抽象类有实现，那么会输出抽象类默认方法。
比如：

![](https://img-blog.csdnimg.cn/img_convert/70a809a1a123d2dfa51f16d7ff7043fd.png)

![](https://img-blog.csdnimg.cn/img_convert/7a8130d6994494b764946162f41a79e1.png)

- 抽象类中可以有具体的方法以及属性（成员变量）
- 抽象类和普通类之间有很多相同的地方，比如他们都可以都静态成员与静态代码块等等。

## 2.接口怎么定义和实现？
> * 接口就是对方法或者动作的抽象，比如`person.class`想要成为教师，可以实现教师的接口，可以理解为增加能力。
> * 接口不允许定义没有初始化的属性变量，可以定义`public static final int i=5;`,以及`public int number =0;`,但不允许`public int num;`这样定义，所有`private`的变量都不允许出现，下面是图片


![](https://img-blog.csdnimg.cn/img_convert/b4c8b2a8b04f17b68cb33c868f0455df.png)


定义`public int number =0;`默认是final修饰的，所以也不能改变它的值:
![](https://img-blog.csdnimg.cn/img_convert/74bb4bf81789e6856d977e747718b883.png)

下面是正确的接口代码：Teacher.java
```java
public interface Teacher {
    public static final int i=5;
    public int number =0;
    public void teach();
    void study();
}
```
- 实现类TeacherClass.java
```java
public class TeacherClass implements Teacher{
    @Override
    public void teach() {
        System.out.println("我是一名老师，我要教书");
        System.out.println("接口的static int是："+i);
    }

    @Override
    public void study() {
        System.out.println("我是一名老师，我也要学习");
        System.out.println("接口的int number是："+number);
    }
}
```
- 测试类Test.java
```java
public class Test {
    public static void main(String[] args){
        TeacherClass teacherClass = new TeacherClass();
        teacherClass.study();
        teacherClass.teach();
        System.out.println("-----------------------------------------------------");
        Teacher teacher =teacherClass;
        teacher.study();
        teacher.teach();
    }
}
```
结果：
![](https://img-blog.csdnimg.cn/img_convert/ba958c533602f309ee63be73b4f76c8b.png)

分析：接口里面所定义的成员变量都是`final`的，不可变的，实现接口必须实现接口里面所有的方法，不能只实现一部分，没有使用`static final`修饰的，默认也是`final`，同时必须有初始化的值，接口不能直接创建对象，比如~~Teacher teacher = new Teacher()~~ ，但是可以先创建一个接口的实现类，然后再赋值于接口对象。

## 3.总结与对比

抽象类 | 接口
---|---
使用关键字`abstract`修饰 | 使用关键字`interface`
使用关键字`extends`实现继承，可以只实现一部分方法，一部分不实现，或者不实现也可以 | `implements`来实现接口，实现接口必须实现里面都有的方法
抽象类里面的方法可以是空实现，可以默认实现，但是必须要带{} | 接口里面的方法都没有实现体，也就是{}
抽象类中可以有具体的方法以及属性，也可以有静态代码块，静态成员变量 | 接口里面不能有普通成员变量，必须都是不可变的`final`成员变量，而且所有的成员变量都必须是`public`
抽象类里面的方法可以是`public`,`protect`,`private`,但是`private`无法继承，所以很少人会这么写，如果没有修饰符，那么只能是同一个包下面的类才能继承 | 接口的方法只能是`public`或者无修饰符，所有的`private`修饰都是会报错的
如果有改动，添加新的方法，可以直接在抽象类中实现默认的即可，也可以在实现类中实现 | 接口增加新方法必须在接口中声明，然后在实现类中进行实现
抽象类不能直接创建对象| 接口也不能直接创建对象 ，可以赋予实现类的对象
抽象类可以有`main`方法，而且我们可以直接运行，抽象类也可以有构造器 | 接口不能有`main`方法，接口不能有构造器

那么我们什么时候使用接口什么时候使用抽象类呢？
> * java有一个缺点，只能实现单继承，个人觉得接口是为了弥补单继承而设计的。
> * 接口是对本质的抽象，比如人，可以设计为`person.class`这个抽象类，提供相关的方法，属性，但是接口是只提供方法的，也就是像增加功能的，那么也就是对方法的抽象。
> * 如果需要默认实现，或者基本功能不断改变，那么建议使用抽象类，如果只是增加一种方法，那么建议使用接口，如果想实现多重继承，只能是接口与抽象类一起使用以达到想要实现的功能。

本文章是初学时的记录，仅是初级的对比，深入学习还需各位保持Keep going~

**此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~**

**技术之路不在一时，山高水长，纵使缓慢，驰而不息。Keep going~**

**公众号：秦怀杂货店**

![](https://img-blog.csdnimg.cn/img_convert/7d98fb66172951a2f1266498e004e830.png)
