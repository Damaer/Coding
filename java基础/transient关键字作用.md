[TOC]
# 1.从Serilizable说到transient
我们知道，如果一个对象需要序列化，那么需要实现`Serilizable`接口，那么这个类的所有非静态属性，都会被序列化。

注意：上面说的是**非静态属性**，因为静态属性是属于类的，而不是属于类对象的，而序列化是针对类对象的操作，所以这个根本不会序列化。下面我们可以实验一下：
实体类`Teacher.class`:
```java
import java.io.Serializable;

class Teacher implements Serializable {
    public int age;
    public static String SchoolName;

    public Teacher(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "age=" + age +
                '}';
    }
}
```
测试代码`SerialTest.java`,基本思路就是初始化的时候，静态属性`SchoolName`为"东方小学",序列化对象之后，将静态属性修改，然后，反序列化，发现其实静态变量还是修改之后的，说明静态变量并没有被序列化。
```java
import java.io.*;

public class SerialTest {
    public static void main(String[] args) {
        Teacher.SchoolName = "东方小学";
        serial();
        Teacher.SchoolName = "西方小学";
        deserial();
        System.out.println(Teacher.SchoolName);
    }
    // 序列化
    private static void serial(){
        try {
            Teacher teacher = new Teacher(9);
            FileOutputStream fileOutputStream = new FileOutputStream("Teacher.txt");
            ObjectOutputStream objectOutputStream= new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(teacher);
            objectOutputStream.flush();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }
    // 反序列化
    private static void deserial() {
        try {
            FileInputStream fis = new FileInputStream("Teacher.txt");
            ObjectInputStream ois = new ObjectInputStream(fis);
            Teacher teacher = (Teacher) ois.readObject();
            ois.close();
            System.out.println(teacher.toString());
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
输出的结果，证明**静态变量没有被序列化！！！**
```java
Teacher{age=9}
西方小学
```
# 2.序列化属性对象的类需要实现`Serilizable`接口？
突然想到一个问题，如果有些属性是对象，而不是基本类型，需不需要改属性的类型也实现`Serilizable`呢？

**问题的答案是：需要！！！**

下面是实验过程：

首先，有一个`Teacher.java`,实现了`Serializable`,里面有一个属性是`School`类型：
```java
import java.io.Serializable;

class Teacher implements Serializable {
    public int age;

    public School school;
    public Teacher(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "age=" + age +
                '}';
    }
}
```

`School`类型,不实现`Serializable`:
```java

public class School {
    public String name;

    public School(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

测试代码，我们只测试序列化：
```java
import java.io.*;

public class SerialTest {
    public static void main(String[] args) {
        serial();
    }

    private static void serial(){
        try {
            Teacher teacher = new Teacher(9);
            teacher.school = new School("东方小学");
            FileOutputStream fileOutputStream = new FileOutputStream("Teacher.txt");
            ObjectOutputStream objectOutputStream= new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(teacher);
            objectOutputStream.flush();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

}
```
会发现报错了，报错的原因是：**School不能被序列化，也就是没有实现序列化接口**，所以如果我们想序列化一个对象，那么这个对象的属性也必须是可序列化的，或者**它是`transient`**修饰的。
```java
java.io.NotSerializableException: com.aphysia.transienttest.School
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1548)
	at java.io.ObjectOutputStream.writeSerialData(ObjectOutputStream.java:1509)
	at java.io.ObjectOutputStream.writeOrdinaryObject(ObjectOutputStream.java:1432)
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1178)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
	at com.aphysia.transienttest.SerialTest.serial(SerialTest.java:18)
	at com.aphysia.transienttest.SerialTest.main(SerialTest.java:9)
```
当我们将`School`实现序列化接口的时候，发现一切就正常了...问题完美解决


# 3.不想被序列化的字段怎么办？
但是如果有一个变量不是静态变量，但是我们也不想序列化它，因为它可能是一些密码等敏感的字段，或者它是不那么重要的字段，我们不希望增加报文大小，所以想在序列化报文中排除该字段。或者改字段存的是引用地址，不是真正重要的数据，比如`ArrayList`里面的`elementData`。


这个时候就需要使用`transient` 关键字，将改字段屏蔽。

当我们用`transient`修饰`School`的时候：
```java
import java.io.Serializable;

class Teacher implements Serializable {
    public int age;

    public transient School school;
    public Teacher(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "age=" + age +
                ", school=" + school +
                '}';
    }
}
```

```java
import java.io.Serializable;

public class School implements Serializable {
    public String name;

    public School(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

执行下面序列化和反序列化的代码：
```java
import java.io.*;

public class SerialTest {
    public static void main(String[] args) {
        serial();
        deserial();
    }

    private static void serial(){
        try {
            Teacher teacher = new Teacher(9);
            teacher.school = new School("东方小学");
            FileOutputStream fileOutputStream = new FileOutputStream("Teacher.txt");
            ObjectOutputStream objectOutputStream= new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(teacher);
            objectOutputStream.flush();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

    private static void deserial() {
        try {
            FileInputStream fis = new FileInputStream("Teacher.txt");
            ObjectInputStream ois = new ObjectInputStream(fis);
            Teacher teacher = (Teacher) ois.readObject();
            ois.close();
            System.out.println(teacher.toString());
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

执行结果如下，可以看到`teacher`字段反序列化出来，其实是null，这也是`transient`起作用了。
但是注意，`transient`只能修饰变量，但是不能修饰类和方法，

# 4.`ArrayList`里面的`elementData`都被`transient` 关键字修饰了，为什么`ArrayList`还可以序列化呢？
这里提一下，既然`transient`修饰了`ArrayList`的数据节点，那么为什么序列化的时候我们还是可以看到`ArrayList`的数据节点呢？
这是因为序列化的时候：
> 如果仅仅实现了`Serializable`接口，那么序列化的时候，肯定是调用`java.io.ObjectOutputStream.defaultWriteObject()`方法，将对象序列化。然后如果是`transient`修饰了该属性，肯定该属性就不能序列化。
> 但是，如果我们虽然实现了`Serializable`接口,也`transient`修饰了该属性,该属性确实不会在默认的`java.io.ObjectOutputStream.defaultWriteObject()`方法里面被序列化了，但是我们可以重写一个`writeObject()`方法，这样一来，序列化的时候调用的就是`writeObject()`，而不是`java.io.ObjectOutputStream.defaultWriteObject()`。

下面的源码是`ObjectInputStream.writeObject(Object obj)`,里面底层其实会有反射的方式调用到重写的对象的`writeObject()`方法，这里不做展开。
```java
    public final void writeObject(Object obj) throws IOException {
        // 如果可以被重写，那么就会调用重写的方法
        if (enableOverride) {
            writeObjectOverride(obj);
            return;
        }
        try {
            writeObject0(obj, false);
        } catch (IOException ex) {
            if (depth == 0) {
                writeFatalException(ex);
            }
            throw ex;
        }
    }
```

`ArrayList`重写的`writeOject()`方法如下：
```java
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        // 默认的序列化对象的方法
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            // 序列化对象的值
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

我们可以看到，`writeOject()`里面其实在里面调用了默认的方法`defaultWriteObject()`，`defaultWriteObject()`底层其实是调用改了`writeObject0()`。`ArrayList`重写的`writeOject()`的思路主要是先序列化默认的，然后序列化数组大小，再序列化数组`elementData`里面真实的元素。这就达到了序列化元素真实内容的目的。

# 5.除了transient，有没有其他的方式，可以屏蔽反序列化？
且慢，问出这个问题，答案肯定是有的！！！那就是**Externalizable**接口。

具体情况：`Externalizable`意思就是，类里面有很多很多属性，但是我只想要一部分，要屏蔽大部分，那么我不想在大部分的属性前面加关键字`transient`，我只想标识一下自己序列化的字段，这个时候就需要使用`Externalizable`接口。

show me the code!

首先定义一个`Person.java`,里面有三个属性
- age:年龄
- name：名字（被`transient`修饰）
- score：分数

实现了`Externalizable`接口，就必须实现`writeExternal()`和`readExternal()`方法。

- writeExternal：将需要序列化的属性进行自定义序列化
- readExternal：将需要反序列化的属性进行自定义反序列化

```java
import java.io.Externalizable;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectOutput;

public class Person implements Externalizable {
    public int age;
    public transient String name;
    public int score;

    // 必须实现无参构造器
    public Person() {
    }

    public Person(int age, String name, int score) {
        this.age = age;
        this.name = name;
        this.score = score;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        /*
         * 指定序列化时候写入的属性。这里不写入score
         */
        out.writeObject(age);
        out.writeObject(name);
        out.writeObject(score);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        /*
         * 指定序列化时候写入的属性。这里仍然不写入年龄
         */
        this.age = (int)in.readObject();
        this.name = (String)in.readObject();
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", score='" + score + '\'' +
                '}';
    }
}

```
上面的代码，我们可以看出，序列化的时候，将三个属性都写进去了，但是反序列化的时候，我们仅仅还原了两个，那么我们来看看测试的代码：
```java
import java.io.*;

public class ExternalizableTest {
    public static void main(String[] args) {
        serial();
        deserial();
    }

    private static void serial(){
        try {
            Person person = new Person(9,"Sam",98);
            FileOutputStream fileOutputStream = new FileOutputStream("person.txt");
            ObjectOutputStream objectOutputStream= new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(person);
            objectOutputStream.flush();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

    private static void deserial() {
        try {
            FileInputStream fis = new FileInputStream("person.txt");
            ObjectInputStream ois = new ObjectInputStream(fis);
            Person person = (Person) ois.readObject();
            ois.close();
            System.out.println(person);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

测试结果如下,就可以发现其实前面两个都反序列化成功了，后面那个是因为我们重写的时候，没有自定义该属性的反序列化，所以没有是正常的啦...
```java
Person{age=9, name='Sam', score='0'}
```

如果细心点，可以发现，有一个字段是`transient`修饰的，不是说修饰了，就不会被序列化么，怎么序列化出来了。

没错，只要实现了`Externalizable`接口，其实就不会被`transient`左右了，只会按照我们自定义的字段进行序列化和反序列化，这里的`transient`是无效的...

关于序列化的`transient`暂时到这，keep going~

**【作者简介】**：  
秦怀，公众号【**秦怀杂货店**】作者，技术之路不在一时，山高水长，纵使缓慢，驰而不息。这个世界希望一切都很快，更快，但是我希望自己能走好每一步，写好每一篇文章，期待和你们一起交流。

此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者核实删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~ 


![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201012000828.png)