Lambda在`jdk1.8`里面已经很好用了，在这里不讲底层的实现，只有简单的用法，会继续补全。
首先一个list我们要使用`lambda`的话，需要使用它的`stream()`方法，获取流，才能使用后续的方法。
#### 基础类User.java
```java
public class User {

  public long userId;

  public User() {
  }

  public User(long userId, String name, int age) {
    this.userId = userId;
    this.name = name;
    this.age = age;
  }

  public String name;
  public int age;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  public long getUserId() {
    return userId;
  }

  public void setUserId(long userId) {
    this.userId = userId;
  }

  @Override
  public String toString() {
    return "User{" +
        "name='" + name + '\'' +
        ", age=" + age +
        ", userId=" + userId +
        '}';
  }

  public void output() {
    System.out.println("User{" +
        "name='" + name + '\'' +
        ", age=" + age +
        ", userId=" + userId +
        '}');
  }
}
```

#### 1.遍历元素
使用foreach方法，其中`s->`里面的s指list里面的每一个元素，针对每一个元素都执行后续的方法。如果里面只有一句话，可以直接缩写`foreach(n -> System.out.println(n));`，如果需要执行的方法里面有两句或者多句需要执行的话，需要可以使用`list.stream().forEach(s -> {System.out.println(s);});`形式。
```java
  // 遍历list(String)和对象
  public static void foreachListString() {
    List features = Arrays.asList("Lambdas", "Default Method", "Stream API",
        "Date and Time API");
    features.forEach(n -> System.out.println(n));
    List<String> list = new ArrayList<>();
    list.add("a");
    list.add("b");
    list.add("c");
    // s代表的是里面的每一个元素，{}里面就是每个元素执行的方法，这个比较容易理解
    list.stream().forEach(s -> {
      System.out.println(s);
    });

    // 处理对象
    List<User> users = new ArrayList<>();
    User user1 = new User();
    user1.setAge(1);
    user1.setName("user1");
    user1.setUserId(1);
    users.add(user1);
    users.stream().forEach(s -> s.output());
  }
```
#### 2.转化里面的每一个元素
map是需要返回值的，s代表里面的每一个元素，return 处理后的返回值
```java
public static void mapList() {
    List<String> list = new ArrayList<>();
    list.add("a");
    list.add("b");
    list.add("c");
    List<String> list2 = new ArrayList<>();
    // map代表从一个转成另一个，s代表里面的每一个值，{}代表针对每一个值的处理方法，如果是代码句子，则需要有返回值
    // 返回值代表转化后的值，以下两种都可以
    list2 = list.stream().map(s -> {
      return s.toUpperCase();
    }).collect(Collectors.toList());
    list2.stream().forEach(s -> {
      System.out.println(s);
    });
    list2 = list.stream().map(s -> s.toUpperCase()).collect(Collectors.toList());
    list2.stream().forEach(s -> {
      System.out.println(s);
    });
  }
```

#### 3.条件过滤筛选
使用filter函数，里面的表达式也是需要返回值的，返回值应该为boolean类型，也就是符合条件的就保留在list里面，不符合条件的就被过滤掉。

```java
  // filter过滤
  public static void filterList() {
    List<String> list1 = new ArrayList<>();
    List<String> list2 = new ArrayList<>();
    list1.add("aasd");
    list1.add("agdfs");
    list1.add("bdfh");
    list2 = list1.stream().filter(s -> {
      return s.contains("a");
    }).collect(Collectors.toList());
    list2.stream().forEach(s -> {
      System.out.println(s);
    });
  }
```

#### 4.取出list里面的对象中的元素，返回一个特定的list
这个可以让我们取出list集合中的某一个元素，也是使用map即可。
```java
  // list集合中取出某一属性
  public static void getAttributeList() {
    List<User> list = new ArrayList<>();
    User user1 = new User();
    user1.setUserId(1);
    user1.setName("James");
    user1.setAge(13);
    list.add(user1);
    User user2 = new User();
    user2.setUserId(2);
    user2.setName("Tom");
    user2.setAge(21);
    list.add(user2);
    // 两种书写方式都可以，一个是map里面，使用每一个实例调用User类的getName方法返回值就是转化后的值。
    List<String> tableNames = list.stream().map(User::getName).collect(Collectors.toList());
    tableNames.stream().forEach(s -> {
      System.out.println(s);
    });
    List<String> tableNames1 = list.stream().map(u -> u.getName()).collect(Collectors.toList());
    tableNames1.stream().forEach(s -> {
      System.out.println(s);
    });
  }
```

#### 5.分组

可以根据某一个属性来分组，获得`map`

``` java
  // 分组,每一组都是list
  public static void groupBy() {
    List<User> userList = new ArrayList<>();// 存放user对象集合
    User user1 = new User(1, "张三", 24);
    User user2 = new User(2, "李四", 27);
    User user3 = new User(3, "王五", 21);
    User user4 = new User(4, "张三", 22);
    User user5 = new User(5, "李四", 20);
    User user6 = new User(6, "王五", 28);
    userList.add(user1);
    userList.add(user2);
    userList.add(user3);
    userList.add(user4);
    userList.add(user5);
    userList.add(user6);
    //根据name来将userList分组
    Map<String, List<User>> groupBy = userList.stream().collect(Collectors.groupingBy(User::getName));
    System.out.println(groupBy);
  }
```

#### 6.对某一个属性进行求和
比如我们需要对年龄进行求和，可以使用mapToInt()，里面参数应该使用`类名：方法名`，最后需要使用sum()来求和。
```java
public static void getSum(){
    List<User> userList = new ArrayList<>();//存放user对象集合

    User user1 = new User(1, "qw", 24);
    User user2 = new User(2, "qwe", 27);
    User user3 = new User(3, "aasf", 21);
    User user4 = new User(4, "fa", 22);
    User user5 = new User(5, "sd", 20);
    User user6 = new User(6, "yr", 28);

    userList.add(user1);
    userList.add(user2);
    userList.add(user3);
    userList.add(user4);
    userList.add(user5);
    userList.add(user6);
    // sum()方法，则是对每一个元素进行加和计算
    int totalAge = userList.stream().mapToInt(User::getAge).sum();
    System.out.println("和：" + totalAge);
  }
```
#### 7.将list转化成map
比如我们需要list里面的对象的id和这个对象对应，那就是需要转换成map。需要在collect()方法里面使用Collectors的toMap()方法即可，参数就是key和value。

```java
 public static void listToMap(){
    List<User> userList = new ArrayList<>();

    User user1 = new User(1, "12", 22);
    User user2 = new User(2, "21", 17);
    User user3 = new User(3, "a", 11);
    User user4 = new User(4, "a", 22);
    User user5 = new User(5, "af", 22);
    User user6 = new User(6, "fa", 25);

    userList.add(user1);
    userList.add(user2);
    userList.add(user3);
    userList.add(user4);
    userList.add(user5);
    userList.add(user6);

    Map<Long,User> userMap = userList.stream().collect(Collectors.toMap(User::getUserId, user -> user));
    System.out.println("toMap：" + userMap.toString());
  }
```

**此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~**

**技术之路不在一时，山高水长，纵使缓慢，驰而不息。**

**公众号：秦怀杂货店**

![](https://img-blog.csdnimg.cn/img_convert/7d98fb66172951a2f1266498e004e830.png)