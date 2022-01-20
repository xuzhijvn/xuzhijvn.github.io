---
title: "Java switch表达式支持的数据类型"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Java"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> "# Java switch表达式支持的数据类型"



```java
switch(expression){
    case value :
       //语句
       break; //可选
    case value :
       //语句
       break; //可选
    //你可以有任意数量的case语句
    default : //可选
       //语句
}
```

这里的 expression 支持：

1、基本数据类型：byte, short, char, int

2、包装数据类型：Byte, Short, Character, Integer

3、枚举类型：Enum

4、 字符串类型：String（Jdk 7+ 开始支持）

为什么不支持long、float、double数据类型？

switch 底层是使用 int 型 来进行判断的，即使是枚举、String类型，最终也是转变成 int 型。由于 long、float、double 型表示范围大于 int 型，因此不支持 long、float、double 类型。 （String类型最终是转成了int类型的hashCode；枚举最终转成了枚举对象的定义顺序，即 ordinal值）

下面举一个使用包装类型和枚举的，其实也不难，注意只能用在 switch 块里面

```java
// 使用包装类型
Integer value = 5;
switch (value) {
    case 3:
        System.out.println("3");
        break;
    case 5:
        System.out.println("5");
        break;
    default:
        System.out.println("default");
}

// 使用枚举类型
Status status = Status.PROCESSING;
switch (status) {
    case OPEN:
        System.out.println("open");
        break;
    case PROCESSING:
        System.out.println("processing");
        break;
    case CLOSE:
        System.out.println("close");
        break;
    default:
        System.out.println("default");
}
```

