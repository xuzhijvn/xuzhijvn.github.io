+++
title = "Java序列化"
description = ""
date = 2021-09-27T11:24:13+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "编程思想"
]
tags = [
  "Java"
]
series = ["Manual"]
images = []

+++



1. 对象的类名、实例变量（包括基本类型，数组，对其他对象的引用）都会被序列化；方法、类变量、transient实例变量都不会被序列化；
2. Serializable反序列化不会调用构造方法；
3. 单例类序列化，需要重写readResolve()方法；否则会破坏单例原则；
4. 序列化对象的`引用类型变量`也要实现Serializable接口；
5. 同一对象序列化多次，只有第一次序列化为二进制流，以后都只是保存序列化编号，不会重复序列化；
6. 使用Externalizable需要实现它的接口，并提供无参构造方法。

<!--more-->



### 1. 反序列化不调用构造方法

反序列的对象是由JVM自己生成的对象，不通过构造方法生成。

Person：

```java
 public class Person implements Serializable {
     private String name;
     private int age;
   	
   	 //省略setter getter

     public Person(String name, int age) {
         System.out.println("反序列化，你调用我了吗？");
         this.name = name;
         this.age = age;
     }
     @Override
     public String toString() {
         return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
     }
 }
```

测试：

```java
public class Test2 {
    public static void main(String[] args) throws Exception {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt")); ObjectInputStream ios = new ObjectInputStream(new FileInputStream("person.txt"))) {
            Person person = new Person("hello tony", 18);
            oos.writeObject(person);
            Person obj = (Person) ios.readObject();
            System.out.println(obj);
        }
    }
}
```

结果：

```json
反序列化，你调用我了吗？
Person{name='hello tony', age=18}
```

### 2. 自定义序列、反序列化策略

我可以使用 `transient` 关键字修饰不需要序列化的字段，但是这种自定义方式较弱。

`writeObject` 和 `readObject` 是一对，`writeReplace` 和 `readResolve` 是一对

利用这几个函数可以自定义序列化、反序列化策略

执行顺序为：

writeReplace 👉  writeObject 👉  readObject 👉  readResolve

Person：

修改之前的Person如下

```java
 public class Person implements Serializable {
     private String name;
     private int age;

     public Person(String name, int age) {
         System.out.println("反序列化，你调用我了吗？");
         this.name = name;
         this.age = age;
     }
   
     //省略setter getter

     private Object writeReplace() throws ObjectStreamException {
         System.out.println("writeReplace");
         return new Person("tony", 20);
     }

     private Object readResolve() throws ObjectStreamException{
         System.out.println("readResolve");
         return new Person("tony123", 123);
     }

     private void writeObject(ObjectOutputStream out) throws IOException {
         System.out.println("writeObject");
         //将名字反转写入二进制流
         out.writeObject(new StringBuffer(this.name).reverse());
         out.writeInt(age);
     }

     private void readObject(ObjectInputStream ins) throws IOException, ClassNotFoundException {
         System.out.println("readObject");
         //将读出的字符串反转恢复回来
         this.name = ((StringBuffer)ins.readObject()).reverse().toString();
         this.age = ins.readInt();
     }

     @Override
     public String toString() {
         return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
     }
 }
```

测试：

测试代码不变

```java
public class Test2 {
    public static void main(String[] args) throws Exception {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt")); ObjectInputStream ios = new ObjectInputStream(new FileInputStream("person.txt"))) {
            Person person = new Person("hello tony", 18);
            oos.writeObject(person);
            Person obj = (Person) ios.readObject();
            System.out.println(obj);
        }
    }
}
```

结果：

```json
反序列化，你调用我了吗？
writeReplace
反序列化，你调用我了吗？
writeObject
readObject
readResolve
反序列化，你调用我了吗？
Person{name='tony123', age=123}
```

### 3. 引用类型成员变量序列化

如果一个可序列化的类的成员不是基本类型，也不是String类型，那这个引用类型也必须是可序列化的；否则，会导致此类不能序列化。

### 4.同一个对象只序列化一次

由于java序利化算法不会重复序列化同一个对象，只会记录已序列化对象的编号。如果对象内容发生变化，再次序列化，并不会再次将此对象转换为字节序列，而只是保存序列化编号。

测试：

```java
 public class Test1 {
     public static void main(String[] args) {
         try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt"));
              ObjectInputStream ios = new ObjectInputStream(new FileInputStream("person.txt"))) {

             Person person = new Person("tony", 23);
             oos.writeObject(person);

             person.setName("hello tony");
             oos.writeObject(person);

             Person p1 = (Person) ios.readObject();
             Person p2 = (Person) ios.readObject();

             System.out.println(p1 == p2);

             System.out.println(p1);
             System.out.println(p2);

         } catch (Exception e) {
             e.printStackTrace();
         }
     }
 }
```

结果：

```json
反序列化，你调用我了吗？
true
Person{name='tony', age=23}
Person{name='tony', age=23}
```

### 5. Externalizable

Externalizable接口不同于Serializable接口：

- 实现此接口必须实现接口中的`writeExternal`, `readExternal`两个方法实现自定义序列化，这是强制性的；
- 必须提供pulic的无参构造器，因为在反序列化的时候需要反射创建对象。



```java
 public class ExPerson implements Externalizable {
     private String name;
     private int age;

     //注意，必须加上pulic 无参构造器
     public ExPerson() {
     }

     public ExPerson(String name, int age) {
         this.name = name;
         this.age = age;
     }

     @Override
     public String toString() {
         return "ExPerson{" +
                 "name='" + name + '\'' +
                 ", age=" + age +
                 '}';
     }

     @Override
     public void writeExternal(ObjectOutput out) throws IOException {
         //将name反转后写入二进制流
         StringBuffer reverse = new StringBuffer(name).reverse();
         System.out.println(reverse.toString());
         out.writeObject(reverse);
         out.writeInt(age);
     }

     @Override
     public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
         //将读取的字符串反转后赋值给name实例变量
         this.name = ((StringBuffer) in.readObject()).reverse().toString();
         System.out.println(name);
         this.age = in.readInt();
     }
 }
```

测试：

测试方法不变

```java
 public static void main(String[] args) throws IOException, ClassNotFoundException {
     try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ExPerson.txt"));
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream("ExPerson.txt"))) {
         oos.writeObject(new ExPerson("tony", 18));
         ExPerson ep = (ExPerson) ois.readObject();
         System.out.println(ep);
     }
 }
```

虽然Externalizable接口带来了一定的性能提升，但变成复杂度也提高了，所以一般通过实现Serializable接口进行序列化。

## 参考

[java序列化，看这篇就够了](https://juejin.cn/post/6844903848167866375)

