[TOC]

很多时候我们会遇到别人问一个问题：你给我讲一下反射，到底是什么东西？怎么实现的？我们能用反射来做什么？它有什么优缺点？下面我们会围绕着这几个问题展开：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201115212257.png)
## 一、反射机制是什么？
**反射是什么？什么是反？什么是正射？**
有反就有正，我们知道正常情况， 如果我们希望创建一个对象，会使用以下的语句：
```java
Person person = new Person();
```
其实我们第一次执行上面的语句的时候，JVM会先加载`Person.class`，加载到内存完之后，在方法区/堆中会创建了一个`Class`对象,对应这个`Person`类。这里有争议，有人说是在方法区，有些人说是在堆。个人感觉应该JVM规范说是在方法区，但是不是强制要求，而且不同版本的JVM实现也不一样。具体参考以下链接，这里不做解释：
https://www.cnblogs.com/xy-nb/p/6773051.html
而上面正常的初始化对象的方法，也可以说是“正射”,就是使用`Class`对象创建出一个`Person`对象。

而反射则相反，是根据`Person`对象，获取到`Class`对象，然后可以获取到`Person`类的相关信息，进行初始化或者调用等一系列操作。


在**运行状态时**，可以构造任何一个类的对象，获取到任意一个对象所属的类信息，以及这个类的成员变量或者方法，可以调用任意一个对象的属性或者方法。可以理解为具备了 **动态加载对象** 以及 **对对象的基本信息进行剖析和使用** 的能力。

提供的功能包括：
- 1.在运行时判断一个对象所属的类
- 2.在运行时构造任意一个类的对象
- 3.在运行时获取一个类定义的成员变量以及方法
- 4.在运行时调用任意一个对象的方法
- 5.生成动态代理

灵活，强大，可以在运行时装配，无需在组件之间进行源代码链接，但是使用不当效率会有影响。所有类的对象都是Class的实例。
既然我们可以对类的全限定名，方法以及参数等进行配置，完成对象的初始化，那就是相当于增加了java的可配置性。

**这里特别需要明确的一点：类本身也是一个对象，方法也是一个对象，在Java里面万物皆可对象，除了基础数据类型...**

## 二、反射的具体使用
### 2.1 获取对象的包名以及类名
```  java
package invocation;
public class MyInvocation {
    public static void main(String[] args) {
        getClassNameTest();
    }
    
    public static void getClassNameTest(){
        MyInvocation myInvocation = new MyInvocation();
        System.out.println("class: " + myInvocation.getClass());
        System.out.println("simpleName: " + myInvocation.getClass().getSimpleName());
        System.out.println("name: " + myInvocation.getClass().getName());
        System.out.println("package: " +
                "" + myInvocation.getClass().getPackage());
    }
}
```
运行结果：
```  java
class: class invocation.MyInvocation
simpleName: MyInvocation
name: invocation.MyInvocation
package: package invocation
```
由上面结果我们可以看到：
1.`getClass()`:打印会带着class+全类名
2.`getClass().getSimpleName()`：只会打印出类名
3.`getName()`：会打印全类名
4.`getClass().getPackage()`:打印出package+包名

`getClass()`获取到的是一个对象，`getPackage()`也是。

### 2.2 获取Class对象
在java中，一切皆对象。java中可以分为两种对象，实例对象和Class对象。这里我们说的获取Class对象，其实就是第二种，Class对象代表的是每个类在运行时的类型信息，指和类相关的信息。比如有一个`Student`类，我们用`Student student = new Student()`new一个对象出来，这个时候`Student`这个类的信息其实就是存放在一个对象中，这个对象就是**Class类的对象**，而student这个实例对象也会和**Class对象**关联起来。
我们有三种方式可以获取一个类在运行时的Class对象，分别是
- Class.forName("com.Student")
- student.getClass()
- Student.class

实例代码如下：
```java
package invocation;

public class MyInvocation {
    public static void main(String[] args) {
        getClassTest();
    }
    public static void getClassTest(){
        Class<?> invocation1 = null;
        Class<?> invocation2 = null;
        Class<?> invocation3 = null;
        try {
            // 最常用的方法
            invocation1 = Class.forName("invocation.MyInvocation");
        }catch (Exception ex){
            ex.printStackTrace();
        }
        invocation2 = new MyInvocation().getClass();
        invocation3 = MyInvocation.class;
        System.out.println(invocation1);
        System.out.println(invocation2);
        System.out.println(invocation3);
    }
}
```
执行的结果如下,三个结果一样：
```
class invocation.MyInvocation
class invocation.MyInvocation
class invocation.MyInvocation
```

### 2.3 getInstance()获取指定类型的实例化对象
首先我们有一个Student类，后面都会沿用这个类，将不再重复。
```java
class Student{
    private int age;

    private String name;

    public Student() {
    }
    public Student(int age) {
        this.age = age;
    }

    public Student(String name) {
        this.name = name;
    }
    
    public Student(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
```
我们可以使用`getInstance()`方法构造出一个Student的对象：
```java
    public static void getInstanceTest() {
        try {
            Class<?> stduentInvocation = Class.forName("invocation.Student");
            Student student = (Student) stduentInvocation.newInstance();
            student.setAge(9);
            student.setName("Hahs");
            System.out.println(student);

        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
    
    
输出结果如下：
Student{age=9, name='Hahs'}
```
但是如果我们取消不写Student的无参构造方法呢？就会出现下面的报错：
```java
java.lang.InstantiationException: invocation.Student
	at java.lang.Class.newInstance(Class.java:427)
	at invocation.MyInvocation.getInstanceTest(MyInvocation.java:40)
	at invocation.MyInvocation.main(MyInvocation.java:8)
Caused by: java.lang.NoSuchMethodException: invocation.Student.<init>()
	at java.lang.Class.getConstructor0(Class.java:3082)
	at java.lang.Class.newInstance(Class.java:412)
	... 2 more
```
这是因为我们重写了构造方法，而且是有参构造方法，如果不写构造方法，那么每个类都会默认有无参构造方法，重写了就不会有无参构造方法了，所以我们调用`newInstance()`的时候，会报没有这个方法的错误。值得注意的是,`newInstance()`是一个无参构造方法。

### 2.4 通过构造函数对象实例化对象
除了`newInstance()`方法之外，其实我们还可以通过构造函数对象获取实例化对象，怎么理解？这里只构造函数对象，而不是构造函数，也就是构造函数其实就是一个对象，我们先获取构造函数对象，当然也可以使用来实例化对象。

可以先获取一个类的所有的构造方法，然后遍历输出：
```java
    public static void testConstruct(){
        try {
            Class<?> stduentInvocation = Class.forName("invocation.Student");
            Constructor<?> cons[] = stduentInvocation.getConstructors();
            for(int i=0;i<cons.length;i++){
                System.out.println(cons[i]);
            }

        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
```    
输出结果：
```java
public invocation.Student(int,java.lang.String)
public invocation.Student(java.lang.String)
public invocation.Student(int)
public invocation.Student()
```
取出一个构造函数我们可以获取到它的各种信息，包括参数，参数个数，类型等等：
```java
    public static void constructGetInstance() {
        try {
            Class<?> stduentInvocation = Class.forName("invocation.Student");
            Constructor<?> cons[] = stduentInvocation.getConstructors();
            Constructor constructors = cons[0];
            System.out.println("name: " + constructors.getName());
            System.out.println("modifier: " + constructors.getModifiers());
            System.out.println("parameterCount: " + constructors.getParameterCount());
            System.out.println("构造参数类型如下：");
            for (int i = 0; i < constructors.getParameterTypes().length; i++) {
                System.out.println(constructors.getParameterTypes()[i].getName());
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
```
输出结果,`modifier`是权限修饰符，1表示为`public`，我们可以知道获取到的构造函数是两个参数的，第一个是int，第二个是String类型，看来获取出来的顺序并不一定是我们书写代码的顺序。
```java
name: invocation.Student
modifier: 1
parameterCount: 2
构造参数类型如下：
int
java.lang.String
```
既然我们可以获取到构造方法这个对象了，那么我们可不可以通过它去构造一个对象呢？**答案肯定是可以！！！**
下面我们用不同的构造函数来创建对象：
```java
    public static void constructGetInstanceTest() {
        try {
            Class<?> stduentInvocation = Class.forName("invocation.Student");
            Constructor<?> cons[] = stduentInvocation.getConstructors();
            // 一共定义了4个构造器
            Student student1 = (Student) cons[0].newInstance(9,"Sam");
            Student student2 = (Student) cons[1].newInstance("Sam");
            Student student3 = (Student) cons[2].newInstance(9);
            Student student4 = (Student) cons[3].newInstance();
            System.out.println(student1);
            System.out.println(student2);
            System.out.println(student3);
            System.out.println(student4);

        } catch (Exception ex) {
            ex.printStackTrace();
        }
```
输出如下：
```java
Student{age=9, name='Sam'}
Student{age=0, name='Sam'}
Student{age=9, name='null'}
Student{age=0, name='null'}
```
构造器的顺序我们是必须一一针对的，要不会报一下的参数不匹配的错误：
```java
java.lang.IllegalArgumentException: argument type mismatch
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at invocation.MyInvocation.constructGetInstanceTest(MyInvocation.java:85)
	at invocation.MyInvocation.main(MyInvocation.java:8)
```

### 2.5 获取类继承的接口
通过反射我们可以获取接口的方法，如果我们知道某个类实现了接口的方法，同样可以做到通过类名创建对象调用到接口的方法。

首先我们定义两个接口，一个`InSchool`:
```  java
public interface InSchool {
    public void attendClasses();
}
```
一个`AtHome`:
```java
public interface AtHome {
    public void doHomeWork();
}
```
创建一个实现两个接口的类`Student.java`
```java
public class Student implements AtHome, InSchool {
    public void doHomeWork() {
        System.out.println("I am a student,I am doing homework at home");
    }

    public void attendClasses() {
        System.out.println("I am a student,I am attend class in school");
    }
}
```
测试代码如下：
```java
public class Test {
    public static void main(String[] args) throws Exception {
        Class<?> studentClass = Class.forName("invocation.Student");
        Class<?>[] interfaces = studentClass.getInterfaces();
        for (Class c : interfaces) {
            // 获取接口
            System.out.println(c);
            // 获取接口里面的方法
            Method[] methods = c.getMethods();
            // 遍历接口的方法
            for (Method method : methods) {
                // 通过反射创建对象
                Student student = (Student) studentClass.newInstance();
                // 通过反射调用方法
                method.invoke(student, null);
            }
        }
    }
}
```
结果如下：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201114003042.png)

可以看出其实我们可以获取到接口的数组，并且里面的顺序是我们继承的顺序，通过接口的**Class对象**，我们可以获取到接口的方法，然后通过方法反射调用实现类的方法，因为这是一个无参数的方法，所以只需要传null即可。

### 2.6 获取父类相关信息
主要是使用`getSuperclass()`方法获取父类，当然也可以获取父类的方法，执行父类的方法,首先创建一个`Animal.java`:
```java
public class Animal {
    public void doSomething(){
        System.out.println("animal do something");
    }
}
```
`Dog.java`继承于`Animal.java`：
```java
public class Dog extends Animal{
    public void doSomething(){
        System.out.println("Dog do something");
    }
}
```
我们可以通过反射创建`Dog`对象，获取其父类`Animal`以及创建对象，当然也可以获取`Animal`的默认父类`Object`：
```java
public class Test {
    public static void main(String[] args) throws Exception {
        Class<?> dogClass = Class.forName("invocation02.Dog");
        System.out.println(dogClass);
        invoke(dogClass);

        Class<?> animalClass = dogClass.getSuperclass();
        System.out.println(animalClass);
        invoke(animalClass);

        Class<?> objectClass = animalClass.getSuperclass();
        System.out.println(objectClass);
        invoke(objectClass);
    }

    public static void invoke(Class<?> myClass) throws Exception {
        Method[] methods = myClass.getMethods();
        // 遍历接口的方法
        for (Method method : methods) {
            if (method.getName().equalsIgnoreCase("doSomething")) {
                // 通过反射调用方法
                method.invoke(myClass.newInstance(), null);
            }
        }
    }
}
```
输入如下：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201114144931.png)
### 2.7 获取当前类的公有属性和私有属性以及更新
创建一个`Person.java`,里面有静态变量，非静态变量，以及`public`，`protected`,`private`不同修饰的属性。
```java
public class Person {

    public static String type ;

    private static String subType ;

    // 名字（公开）
    public String name;

    protected String gender;

    private String address;

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```
使用`getFields()`可以获取到public的属性，包括static属性，使用`getDeclaredFields()`可以获取所有声明的属性，不管是`public`，`protected`,`private`不同修饰的属性。

修改`public`属性,只需要`field.set(object，value)`即可，但是`private`属性不能直接set，否则会报以下的错误。
```java
Exception in thread "main" java.lang.IllegalAccessException: Class invocation03.Tests can not access a member of class invocation03.Person with modifiers "private"
	at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:102)
	at java.lang.reflect.AccessibleObject.slowCheckMemberAccess(AccessibleObject.java:296)
	at java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:288)
	at java.lang.reflect.Field.set(Field.java:761)
	at invocation03.Tests.main(Tests.java:21)
```
那么需要怎么做呢？private默认是不允许外界操作其值的，这里我们可以使用`field.setAccessible(true);`，相当于打开了操作的权限。

那static的属性修改和非static的一样，但是我们怎么获取呢？
如果是`public`修饰的，可以直接用类名获取到，如果是`private`修饰的，那么需要使用`filed.get(object)`,这个方法其实对上面说的所有的属性都可以的。
测试代码如下
```java
public class Tests {
    public static void main(String[] args) throws Exception{
        Class<?> personClass = Class.forName("invocation03.Person");
        Field[] fields = personClass.getFields();
        // 获取公开的属性
        for(Field field:fields){
            System.out.println(field);
        }
        System.out.println("=================");
        // 获取所有声明的属性
        Field[] declaredFields = personClass.getDeclaredFields();
        for(Field field:declaredFields){
            System.out.println(field);
        }
        System.out.println("=================");
        Person person = (Person) personClass.newInstance();
        person.name = "Sam";
        System.out.println(person);

        // 修改public属性
        Field fieldName = personClass.getDeclaredField("name");
        fieldName.set(person,"Jone");

        // 修改private属性
        Field addressName = personClass.getDeclaredField("address");
        // 需要修改权限
        addressName.setAccessible(true);
        addressName.set(person,"东风路47号");
        System.out.println(person);

        // 修改static 静态public属性
        Field typeName = personClass.getDeclaredField("type");
        typeName.set(person,"人类");
        System.out.println(Person.type);

        // 修改静态 private属性
        Field subType = personClass.getDeclaredField("subType");
        subType.setAccessible(true);
        subType.set(person,"黄种人");
        System.out.println(subType.get(person));
    }
}
```

结果：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201114162617.png)

从结果可以看出，不管是`public`，还是`protected`，`private`修饰的，我们都可以通过反射对其进行查询和修改，不管是静态变量还是非静态变量。
`getDeclaredField()`可以获取到所有声明的属性，而`getFields()`则只能获取到`public`的属性。对于非public的属性，我们需要修改其权限才能访问和修改：`field.setAccessible(true)`。

获取属性值需要使用`field.get(object)`，值得注意的是：**每个属性，其本身就是对象**

### 2.8 获取以及调用类的公有/私有方法
既然可以获取到公有属性和私有属性，那么我想，执行公有方法和私有方法应该都不是什么问题？
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201115211213.png)

那下面我们一起来学习一下...

先定义一个类，包含各种修饰符，以及是否包含参数，是否为静态方法，`Person.java`:
```java
public class Person {
    // 非静态公有无参数
    public void read(){
        System.out.println("reading...");
    }

    // 非静态公有无参数有返回
    public String getName(){
        return "Sam";
    }

    // 非静态公有带参数   
    public int readABookPercent(String name){
        System.out.println("read "+name);
        return 80;
    }

    // 私有有返回值
    private String getAddress(){
        return "东方路";
    }

    // 公有静态无参数无返回值
    public static void staticMethod(){
        System.out.println("static public method");
    }

    // 公有静态有参数
    public static void staticMethodWithArgs(String args){
        System.out.println("static public method:"+args);
    }

    // 私有静态方法
    private static void staticPrivateMethod(){
        System.out.println("static private method");
    }
}
```
首先我们来看看获取里面所有的方法：
```  java
public class Tests {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("invocation03.Person");
        Method[] methods = personClass.getMethods();
        for (Method method : methods) {
            System.out.println(method);
        }

        System.out.println("=============================================");
        Method[] declaredMethods = personClass.getDeclaredMethods();
        for (Method method : declaredMethods) {
            System.out.println(method);
        }
    }
}
```

结果如下：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201116000934.png)
咦，我们发现`getMethods()`确实可以获取所有的公有的方法，但是有一个问题，就是他会把父类的也获取到，也就是上面图片绿色框里面的，我们知道所有的类默认都继承了`Object`类，所以它把`Object`的那些方法都获取到了。
而`getDeclaredMethods`确实可以获取到公有和私有的方法，不管是静态还是非静态，但是它是获取不到父类的方法的。


那如果我们想调用方法呢？先试试调用非静态方法：
```java
public class Tests {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("invocation03.Person");
        Person person = (Person) personClass.newInstance();
        Method[] declaredMethods = personClass.getDeclaredMethods();
        for (Method method : declaredMethods) {
            if(method.getName().equalsIgnoreCase("read")){
                method.invoke(person,null);
                System.out.println("===================");
            }else if(method.getName().equalsIgnoreCase("getName")){
                System.out.println(method.invoke(person,null));
                System.out.println("===================");
            }else if(method.getName().equalsIgnoreCase("readABookPercent")){
                System.out.println(method.invoke(person,"Sam"));
                System.out.println("===================");
            }
        }

    }
}
```
结果如下，可以看出`method.invoke(person,null);`是调用无参数的方法，而`method.invoke(person,"Sam")`则是调用有参数的方法，要是有更多参数，也只需要在里面多加一个参数即可，返回值也同样可以获取到。
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201116001756.png)

那么`private`方法呢？我们照着来试试，试试就试试，who 怕 who？
```java
public class Tests {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("invocation03.Person");
        Person person = (Person) personClass.newInstance();
        Method[] declaredMethods = personClass.getDeclaredMethods();
        for (Method method : declaredMethods) {
            if(method.getName().equalsIgnoreCase("getAddress")){
                method.invoke(person,null);
            }
        }

    }
}
```
结果报错了：
```java
Exception in thread "main" java.lang.IllegalAccessException: Class invocation03.Tests can not access a member of class invocation03.Person with modifiers "private"
	at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:102)
	at java.lang.reflect.AccessibleObject.slowCheckMemberAccess(AccessibleObject.java:296)
	at java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:288)
	at java.lang.reflect.Method.invoke(Method.java:491)
	at invocation03.Tests.main(Tests.java:13)
```
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201116002147.png)

一看就是没有权限，小场面，不要慌，我来操作一波,只要加上
```java
method.setAccessible(true);
```
哦豁，完美解决了...
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201116002514.png)


那么问题来了，上面说的都是非静态的，我就想要调用静态的方法。
当然用上面的方法，对象也可以直接调用到类的方法的：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201116002936.png)

一点问题都没有，为什么输出结果有几个null,那是因为这函数是无返回值的呀，笨蛋...

如果我不想用遍历方法的方式，再去判断怎么办？能不能直接获取到我想要的方法啊？那答案肯定是可以啊。
```  java
public class Tests {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("invocation03.Person");
        Person person = (Person) personClass.newInstance();
        Method method = personClass.getMethod("readABookPercent", String.class);
        method.invoke(person, "唐诗三百首");
    }
}
```
结果和上面调用的完全一样，图我就不放了，就一行字。要是这个方法没有参数呢？那就给一个null就可以啦。或者不给也可以。
```java
public class Tests {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("invocation03.Person");
        Person person = (Person) personClass.newInstance();
        Method method = personClass.getMethod("getName",null);
        System.out.println(method.invoke(person));
    }
}
```

## 三、反射的优缺点
### 3.1 优点
反射可以在不知道会运行哪一个类的情况下，获取到类的信息，创建对象以及操作对象。这其实很方便于拓展，所以反射会是框架设计的灵魂，因为框架在设计的时候，为了降低耦合度，肯定是需要考虑拓展等功能的，不能将类型写死，硬编码。

降低耦合度，变得很灵活，在运行时去确定类型，绑定对象，体现了多态功能。

### 3.2 缺点
这么好用，没有缺点？怎么可能！！！有利就有弊，事物都是有双面性的。
即使功能很强大，但是反射是需要动态类型的，`JVM`没有办法优化这部分代码，执行效率相对直接初始化对象较低。一般业务代码不建议使用。

反射可以修改权限，比如上面访问到`private`这些方法和属性，这是会破坏封装性的，有安全隐患，有时候，还会破坏单例的设计。

反射会使代码变得复杂，不容易维护，毕竟代码还是要先写给人看的嘛，逃~

**【作者简介】**：  
秦怀，公众号【**秦怀杂货店**】作者，技术之路不在一时，山高水长，纵使缓慢，驰而不息。这个世界希望一切都很快，更快，但是我希望自己能走好每一步，写好每一篇文章，期待和你们一起交流。

此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者核实删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~ 


![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201012000828.png)
