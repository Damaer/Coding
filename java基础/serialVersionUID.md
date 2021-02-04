[TOC]
# 正常不设置serialVersionUID 的序列化和反序列化
先定义一个实体`Student.class`,**需要实现`Serializable`接口，但是不需要实现get()，set()方法**
```java

import java.io.Serializable;
public class Student implements Serializable {
    private int age;
    private String name;
    public Student(int age, String name) {
        this.age = age;
        this.name = name;
    }
    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}
```
测试类，思路是先把Student对象序列化到`Student.txt`文件，然后再讲`Student.txt`文件反序列化成对象，输出。
```java
public class SerialTest {
    public static void main(String[] args) {
        serial();
        deserial();
    }
    // 序列化
    private static void serial(){
        Student student = new Student(9, "Mike");
        try {
            FileOutputStream fileOutputStream = new FileOutputStream("Student.txt");
            ObjectOutputStream objectOutputStream= new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(student);
            objectOutputStream.flush();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }
    // 反序列化
    private static void deserial() {
        try {
            FileInputStream fis = new FileInputStream("Student.txt");
            ObjectInputStream ois = new ObjectInputStream(fis);
            Student student = (Student) ois.readObject();
            ois.close();
            System.out.println(student.toString());
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
输出结果，序列化文件我们可以看到`Student.txt`,反序列化出来，里面的字段都是不变的，说明反序列化成功了。
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201025231938.png)

# 序列化之后，类文件增加了字段，反序列化会怎么样？
先说结果，会失败！！！

我们在`Student.java`中增加了一个属性`score`,重新生成了`toString()`方法。
```java
package com.aphysia.normal;

import java.io.Serializable;

public class Student implements Serializable {
    private int age;
    private String name;
    private int score;
    public Student(int age, String name) {
        this.age = age;
        this.name = name;
    }
    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", score=" + score +
                '}';
    }
}
```
然后重新调用`deserial()`方法，会报错：
```  java
java.io.InvalidClassException: com.aphysia.normal.Student; local class incompatible: stream classdesc serialVersionUID = 7488921480006384819, local class serialVersionUID = 6126416635811747983
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1963)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1829)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2120)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1646)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:482)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:440)
	at com.aphysia.normal.SerialTest.deserial(SerialTest.java:26)
	at com.aphysia.normal.SerialTest.main(SerialTest.java:7)
```

从上面的报错信息中，我们可以看到，类型不匹配，主要是因为`serialVersionUID`变化了！！！

🙋‍♂️🙋‍♂️ **提问环节：我都没有设置`serialVersionUID`,怎么变化的？？？小小的脑袋很多问号**🤔🤔

正是因为没有设置，所以变化了，因为我们增加了一个字段`score`,如果我们不设置`serialVersionUID`，系统就会自动生成，自动生成有风险，就是我们的字段类型或者长度改变（新增或者删除的时候），自动生成的`serialVersionUID`会发生变化，那么以前序列化出来的对象，反序列化的时候就会失败。

实测：序列化完成之后，如果原类型字段减少，不指定`serialVersionUID`的情况下，也是会报不一致的错误。

> 《阿里巴巴 Java 开发手册》中规定，在兼容性升级中，在修改类的时候，不要修改serialVersionUID的原因。除非是完全不兼容的两个版本。所以，serialVersionUID其实是验证版本一致性的。

# 指定`serialVersionUID`,减少或者增加字段会发生什么？
我们生成一个`serialVersionUID`,方法：https://blog.csdn.net/Aphysia/article/details/80620804。

```java
    private static final long serialVersionUID = 7488921480006384819L;
```
然后执行序列化，序列化出文件`Student.txt`后，增加一个字段`score`，执行反序列化。
是可以成功的！！！只是新增的字段是默认值0。
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201026003700.png)

所以今后考虑到迭代的问题的时候，一般可能增加字段或者减少字段，都是需要考虑兼容问题的，所以最好是自己指定`serialVersionUID`，而不是由系统自动生成。自动生成的，由于类文件变化，它也会发生变化，就会出现不一致的问题，导致反序列化失败。

实测：如果我减少了字段，只要指定了`serialVersionUID`，也不会报错！！！


# serialVersionUID生成以及作用？
`serialVersionUID`是为了兼容不同版本的，在JDK中，可以利用JDK的`bin`目录下的`serialver.exe`工具产生这个`serialVersionUID`，对于`Student.class`，执行命令：`serialver Student`。

IDEA生成实际上也是调用这个命令，代码调用可以这样写：
```  java
ObjectStreamClass c = ObjectStreamClass.lookup(Student.class);
long serialID = c.getSerialVersionUID();
System.out.println(serialID);
```
如果不显示的指定，那么不同JVM之间的移植可能也会出错，因为不同的编译器，计算这个值的策略可能不同，计算类没有修改，也会出现不一致的问题。
`getSerialVersionUID()`源码如下：
```java
    public long getSerialVersionUID() {
        // REMIND: synchronize instead of relying on volatile?
        if (suid == null) {
            suid = AccessController.doPrivileged(
                new PrivilegedAction<Long>() {
                    public Long run() {
                        return computeDefaultSUID(cl);
                    }
                }
            );
        }
        return suid.longValue();
    }
```
可以看到上面是使用了一个内部类的方式，使用特权计算`computeDefaultSUID()`:

```java
    private static long computeDefaultSUID(Class<?> cl) {
        // 代理
        if (!Serializable.class.isAssignableFrom(cl) || Proxy.isProxyClass(cl))
        {
            return 0L;
        }

        try {
            ByteArrayOutputStream bout = new ByteArrayOutputStream();
            DataOutputStream dout = new DataOutputStream(bout);
            // 类名
            dout.writeUTF(cl.getName());

            // 修饰符
            int classMods = cl.getModifiers() &
                (Modifier.PUBLIC | Modifier.FINAL |
                 Modifier.INTERFACE | Modifier.ABSTRACT);

            //  方法
            Method[] methods = cl.getDeclaredMethods();
            if ((classMods & Modifier.INTERFACE) != 0) {
                classMods = (methods.length > 0) ?
                    (classMods | Modifier.ABSTRACT) :
                    (classMods & ~Modifier.ABSTRACT);
            }
            dout.writeInt(classMods);

            if (!cl.isArray()) {
                // 继承的接口
                Class<?>[] interfaces = cl.getInterfaces();
                String[] ifaceNames = new String[interfaces.length];
                for (int i = 0; i < interfaces.length; i++) {
                    ifaceNames[i] = interfaces[i].getName();
                }
                // 接口名
                Arrays.sort(ifaceNames);
                for (int i = 0; i < ifaceNames.length; i++) {
                    dout.writeUTF(ifaceNames[i]);
                }
            }
            // 属性
            Field[] fields = cl.getDeclaredFields();
            MemberSignature[] fieldSigs = new MemberSignature[fields.length];
            for (int i = 0; i < fields.length; i++) {
                fieldSigs[i] = new MemberSignature(fields[i]);
            }
            Arrays.sort(fieldSigs, new Comparator<MemberSignature>() {
                public int compare(MemberSignature ms1, MemberSignature ms2) {
                    return ms1.name.compareTo(ms2.name);
                }
            });
            for (int i = 0; i < fieldSigs.length; i++) {
                // 成员签名
                MemberSignature sig = fieldSigs[i];
                int mods = sig.member.getModifiers() &
                    (Modifier.PUBLIC | Modifier.PRIVATE | Modifier.PROTECTED |
                     Modifier.STATIC | Modifier.FINAL | Modifier.VOLATILE |
                     Modifier.TRANSIENT);
                if (((mods & Modifier.PRIVATE) == 0) ||
                    ((mods & (Modifier.STATIC | Modifier.TRANSIENT)) == 0))
                {
                    dout.writeUTF(sig.name);
                    dout.writeInt(mods);
                    dout.writeUTF(sig.signature);
                }
            }
            // 是否有静态初始化
            if (hasStaticInitializer(cl)) {
                dout.writeUTF("<clinit>");
                dout.writeInt(Modifier.STATIC);
                dout.writeUTF("()V");
            }
            // 构造器
            Constructor<?>[] cons = cl.getDeclaredConstructors();
            MemberSignature[] consSigs = new MemberSignature[cons.length];
            for (int i = 0; i < cons.length; i++) {
                consSigs[i] = new MemberSignature(cons[i]);
            }
            Arrays.sort(consSigs, new Comparator<MemberSignature>() {
                public int compare(MemberSignature ms1, MemberSignature ms2) {
                    return ms1.signature.compareTo(ms2.signature);
                }
            });
            for (int i = 0; i < consSigs.length; i++) {
                MemberSignature sig = consSigs[i];
                int mods = sig.member.getModifiers() &
                    (Modifier.PUBLIC | Modifier.PRIVATE | Modifier.PROTECTED |
                     Modifier.STATIC | Modifier.FINAL |
                     Modifier.SYNCHRONIZED | Modifier.NATIVE |
                     Modifier.ABSTRACT | Modifier.STRICT);
                if ((mods & Modifier.PRIVATE) == 0) {
                    dout.writeUTF("<init>");
                    dout.writeInt(mods);
                    dout.writeUTF(sig.signature.replace('/', '.'));
                }
            }

            MemberSignature[] methSigs = new MemberSignature[methods.length];
            for (int i = 0; i < methods.length; i++) {
                methSigs[i] = new MemberSignature(methods[i]);
            }
            Arrays.sort(methSigs, new Comparator<MemberSignature>() {
                public int compare(MemberSignature ms1, MemberSignature ms2) {
                    int comp = ms1.name.compareTo(ms2.name);
                    if (comp == 0) {
                        comp = ms1.signature.compareTo(ms2.signature);
                    }
                    return comp;
                }
            });
            for (int i = 0; i < methSigs.length; i++) {
                MemberSignature sig = methSigs[i];
                int mods = sig.member.getModifiers() &
                    (Modifier.PUBLIC | Modifier.PRIVATE | Modifier.PROTECTED |
                     Modifier.STATIC | Modifier.FINAL |
                     Modifier.SYNCHRONIZED | Modifier.NATIVE |
                     Modifier.ABSTRACT | Modifier.STRICT);
                if ((mods & Modifier.PRIVATE) == 0) {
                    dout.writeUTF(sig.name);
                    dout.writeInt(mods);
                    dout.writeUTF(sig.signature.replace('/', '.'));
                }
            }

            dout.flush();

            MessageDigest md = MessageDigest.getInstance("SHA");
            byte[] hashBytes = md.digest(bout.toByteArray());
            long hash = 0;
            for (int i = Math.min(hashBytes.length, 8) - 1; i >= 0; i--) {
                hash = (hash << 8) | (hashBytes[i] & 0xFF);
            }
            return hash;
        } catch (IOException ex) {
            throw new InternalError(ex);
        } catch (NoSuchAlgorithmException ex) {
            throw new SecurityException(ex.getMessage());
        }
    }
```

从上面这段源码大致来看，其实这个计算`serialVersionUID`,基本是将类名，属性名，属性修饰符，继承的接口，属性类型，名称，方法，静态代码块等等...这些都考虑进去了，都写到一个`DataOutputStream`中，然后再做`hash运算`，所以说,这个东西得指定啊，不指定的话，稍微一改类的东西，就变了...

而且这个东西指定了，没啥事，不要改！！！除非你确定两个版本就不兼容！！！

**【作者简介】**：  
秦怀，公众号【**秦怀杂货店**】作者，技术之路不在一时，山高水长，纵使缓慢，驰而不息。这个世界希望一切都很快，更快，但是我希望自己能走好每一步，写好每一篇文章，期待和你们一起交流。

此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者核实删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~ 


![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/20201012000828.png)





