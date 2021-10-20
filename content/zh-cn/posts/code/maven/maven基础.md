---
title: "Maven基础"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Maven"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# maven基础"

## 1. 问题

项目开发过程中，经常会遇到jar冲突，然后maven根据自己的规则进行冲突解决，导致项目在运行的过程中报错。

1、maven自动解决依赖冲突的规则是什么？

2、如何查看当前项目的maven的依赖树？

3、如何从依赖树中找到自己预期的版本，是被那个jar给覆盖了？

4、如何人工进行依赖冲突解决，达到使用目的？

## 2. 解决问题

### 2.1 maven自动解决依赖冲突的规则是什么？

#### 2.1.1 第一原则：路径最近者优先

项目A有如下的依赖关系：

A->B->C->X(1.0)

A->D->X(2.0)

则该例子中，X的版本是2.0

#### 2.1.2 第二原则：路径相等，先声明者优先

项目A有如下的依赖关系：

A->B->Y(1.0)

A->C->Y(2.0)

若pom文件中B的依赖坐标先于C进行声明，则最终Y的版本为1.0

### 2.2 如何查看当前项目的maven依赖树？

```sh
//进入项目的pom.xml文件的目录下，运行如下命令
//这个是正常依赖的树
mvn dependency:tree
//这个命令是查看maven是如何解决依赖冲突的依赖树
mvn -Dverbose dependency:tree
//如果想将依赖树打印到指定文件中，则命令如下
mvn -Dverbose  dependency:tree -Doutput=/Users/shangxiaofei/sxfoutput.txt
```

### 3. 如何从依赖树中找到自己预期的版本，是被那个jar给覆盖了？

例子：

![img](https://picgo.6and.ltd/img/645085-20190110173328966-1325435360.png)

![img](https://picgo.6and.ltd/img/645085-20190110182627824-965290088-20211010213928890.png)

 

递归依赖的关系列的算是比较清楚了，每行都是一个jar包，根据缩进可以看到依赖的关系。

最后写着compile的就是编译成功的。

最后写着omitted for duplicate的就是有jar包被重复依赖了，但是jar包的版本是一样的。

最后写着omitted for conflict with xxxx的，说明和别的jar包版本冲突了，而该行的jar包不会被引入。比如上面有一行最后写着omitted for conflict with 3.4.6，那么该行的zookeeper:jar:3.4.8不会被引入，会引入3.4.6版本

最后写着version managed from 2.3 ;omitted for duplicate ,表示最终使用commons-pool2最终会使用2.4.2，拒绝使用<dependencyManagement></dependencyManagement>中声明的2.3版本

最后写着version managed from 1.16.8 ;表示最终使用lombok:jar:1.16.22版本 

### 4. 如何人工进行依赖冲突解决，达到使用目的？

解决重复依赖和冲突的方法：

1，修改pom文件中两个dependency元素的位置。如果两个dependency都引用了一个jar包，但是版本不同，classloader只会加载jar包在pom文件中出现的第一个版本，以后出现的其他版本的jar包会被忽略。

不建议使用该方法，因为引用不同版本的jar包本身就是很危险的。

2，使用<exclusions>标签来去掉某个dependency依赖中的某一个jar包或一堆jar包，<exclusion>中的jar包或者依赖的相关jar包都会被忽略，从而在两个dependency都依赖某个jar包时，可以保证只使用其中的一个。

可以这么写：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.8.3.2</version>
    <exclusions>
        <exclusion>
            <artifactId>guava</artifactId>
            <groupId>com.google.guava</groupId>
        </exclusion>
        <exclusion>
            <artifactId>spring</artifactId>
            <groupId>org.springframework</groupId>
        </exclusion>    
    </exclusions>
</dependency>
```

## 3. 深入学习

### 3.1 maven的依赖基础

```xml
<dependency>    
<groupId>com.alibaba.share</groupId>    
<artifactId>test</artifactId>    
<version>1.4</version>    
</dependency>    
    
依赖库命名规则：    
${groupId.part1}/${groupId.part2}/${version}    
例：com/alibaba/share/1.4    
    
依赖库文件命名规则：    
${artifactId}-${version}-${classifier}.${type}    
例：test-1.4-source.jar    
    
注：classfier即分类器，多数的时候是用不到的，不过有写情况需要，例：    
TestNG强制需要你提供分类器，以区别jdk14和jdk15    
<dependency>      
   <groupId>org.testng</groupId>      
   <artifactId>testng</artifactId>      
   <version>5.7</version>      
   <classifier>jdk15</classifier>      
</dependency>    
```

### 3.2 maven依赖范围 

```xml
<dependency>    
      <groupId>junit</groupId>    
      <artifactId>junit</artifactId>    
      <version>3.8.1</version>    
      <scope>test</scope>    
 </dependency> 
```

上面的scope即约定依赖范围。 
compile：默认值，一直可用，最后会被打包 
provided：编译期间可用，不会被传递依赖，不会被打包。例：依赖于web容器中的提供的一个jar包，在编译的时候需要加入依赖（web容器还没有介入），运行的时候由web容器来提供。 
test：执行单元测试时可用，不会被打包，不会被传递依赖 
runtime：运行和测试时需要，但编译时不需要 
system：不推荐使用 

### 3.3 maven依赖管理 

避免不同子模块中依赖版本冲突 
在父pom中配置依赖 

```xml
<dependencyManagement>      
    <dependencies>      
     <dependency>      
       <groupId>mysql</groupId>      
       <artifactId>mysql-connector-java</artifactId>      
      <version>5.1.2</version>      
     </dependency>      
      ...      
   <dependencies>      
</dependencyManagement>    
```

在子pom中添加依赖 

```xml
<dependencies>      
   <dependency>      
    <groupId>mysql</groupId>      
    <artifactId>mysql-connector-java</artifactId>      
   </dependency>      
</dependencies>    
```

dependencyManagement实际上不会真正引入任何依赖，在子pom中添加之后才会。在父pom中配置了之后，子模块只需使用简单groupId和artifactId就能自动继承相应的父模块依赖配置。如果子pom中定义了version，则覆盖management中的。 

### 3.4 可选依赖

```xml
<dependency>    
      <groupId>mysql</groupId>    
      <artifactId>mysql-connector-java</artifactId>    
      <version>1.5</version>    
      <optional>true</optional>    
</dependency>   
```

[maven可选依赖（Optional Dependencies）和依赖排除（Dependency Exclusions）](https://developer.aliyun.com/article/659868)

### 3.5 依赖版本界限 

要求的依赖版本>=3.8且<4.0 

```xml
<version>[3.8,4.0)</version>   
```

要求的依赖版本<=3.8.1 

```xml
<version>[,3.8.1]</version>    
```

要求必须是3.8.1版本，如果不是的话会构建失败，提示版本冲突。原来的写法<version>3.8.1</version>的意思是所有版本都可以，但最好是3.8.1 

```xml
<version>[3.8.1]</version>   
```

### 3.6 排除依赖 

 *依赖project-a但是排除掉对project-a中引入的project-b的依赖* 

```xml
<dependency>    
  <groupId>org.sonatype.mavenbook</groupId>    
  <artifactId>project-a</artifactId>    
  <version>1.0</version>    
  <exclusions>    
    <exclusion>    
      <groupId>org.sonatype.mavenbook</groupId>    
      <artifactId>project-b</artifactId>    
    </exclusion>    
  </exclusions>    
</dependency>    
```

替换依赖 
直接使用上一步中的排除掉，然后添加要替换进来的依赖就可以了，没有什么特殊的标志来标志这个是替换进来的。 

### 3.7 版本冲突仲裁

```
版本仲裁规则(在maven 2.2.1版本上测试验证) 
• 按照项目总POM的DependencyManager版本声明进行仲裁(覆盖),但无警告。 
• 如无仲裁声明,则按照依赖最短路径确定版本。 
• 若相同路径,有严格区间限定的版本优先。 
• 若相同路径,无版本区间,则按照先入为主原则。 
```

### 3.8 依赖冲突解决办法

#### 3.8.1 定位冲突

查看那些jar包依赖了冲突包的命令

```sh
mvn dependency:tree -Dverbose -Dincludes=被依赖的包
```

刚才吹嘘dependency:tree时，我用到了“无处遁形”，其实有时你会发现简单地用dependency:tree往往并不能查看到所有的传递依赖。不过如果你真的想要看所有的，必须得加一个-Dverbose参数，这时就必定是最全的了。全是全了，但显示出来的东西太多，头晕目眩，有没有好法呢？当然有了，加上Dincludes或者Dexcludes说出你喜欢或讨厌，dependency:tree就会帮你过滤出来：

> Dincludes=org.springframework:spring-tx
>
> 过滤串使用groupId:artifactId:version的方式进行过滤，可以不写全啦，如：
>
> mvn dependency:tree -Dverbose -Dincludes=asm:asm  

就会出来asm依赖包的分析信息：

```
[INFO] --- maven-dependency-plugin:2.1:tree (default-cli) @ ridge-test ---
[INFO] com.ridge:ridge-test:jar:1.0.2-SNAPSHOT
[INFO] +- asm:asm:jar:3.2:compile
[INFO] \- org.unitils:unitils-dbmaintainer:jar:3.3:compile
[INFO]    \- org.hibernate:hibernate:jar:3.2.5.ga:compile
[INFO]       +- cglib:cglib:jar:2.1_3:compile
[INFO]       |  \- (asm:asm:jar:1.5.3:compile - omitted for conflict with 3.2)
[INFO]       \- (asm:asm:jar:1.5.3:compile - omitted for conflict with 3.2)
[INFO] ------------------------------------------------------------------------
```

对asm有依赖有一个直接的依赖(asm:asm:jar:3.2)还有一个传递进入的依赖(asm:asm:jar:1.5.3)

#### 3.8.2 将不想要的传递依赖剪除掉

承上，假设我们不希望asm:asm:jar:1.5.3出现，根据分析，我们知道它是经由org.unitils:unitils-dbmaintainer:jar:3.3引入的，那么在pom.xml中找到这个依赖，做其它的调整：

```xml
    <dependency>  
        <groupId>org.unitils</groupId>  
        <artifactId>unitils-dbmaintainer</artifactId>  
        <version>${unitils.version}</version>  
        <exclusions>  
            <exclusion>  
                <artifactId>dbunit</artifactId>  
                <groupId>org.dbunit</groupId>  
            </exclusion>  
            <!-- 这个就是我们要加的片断 -->  
            <exclusion>  
                <artifactId>asm</artifactId>  
                <groupId>asm</groupId>  
            </exclusion>  
        </exclusions>  
    </dependency>  
```

再分析一下，你可以看到传递依赖没有了：

```
    [INFO]  
    [INFO] --- maven-dependency-plugin:2.1:tree (default-cli) @ ridge-test ---  
    [INFO] com.ridge:ridge-test:jar:1.0.2-SNAPSHOT  
    [INFO] \- asm:asm:jar:3.2:compile  
    [INFO] ------------------------------------------------------------------------  
    [INFO] BUILD SUCCESS  
```

## 4. F&Q

### 4.1 runtime

运行时依赖范围。使用该范围的依赖，只对测试和运行的 classpath 有效，但在编译主代码时是无效的。比如 **JDBC 驱动实现类**，就需要在运行测试和运行主代码时候使用，编译的时候，只需 **JDBC 接口**就行。

简单来说，compile、runtime和provided的区别，需要在执行mvn package命令，且打包格式是war之类（而不是默认的jar）的时候才能看出来。

通过compile和provided引入的jar包，里面的类，你在项目中可以直接import进来用，编译没问题，**但是runtime引入的jar包中的类，项目代码里不能直接用，用了无法通过编译，只能通过反射之类的方式来用。**

通过compile和runtime引入的jar包，会出现在你的项目war包里，而provided引入的jar包则不会。

[maven的scope值runtime是干嘛用的?](https://www.zhihu.com/question/338722003)

[Maven依赖配置和依赖范围](http://c.biancheng.net/view/5005.html)

### 4.2 optional

Hibernate等DAO层框架

[Maven 中<optional>true</optional>和<scope>provided</scope>之间的区别](https://segmentfault.com/a/1190000019266080)

[Maven依赖配置和依赖范围](http://c.biancheng.net/view/5005.html)

### 4.3 maven 解决单继承

[使用import scope解决maven继承（单）问题](https://blog.csdn.net/mn960mn/article/details/50894022)

## 参考

[maven查看项目依赖并解决依赖冲突的问题](https://www.cnblogs.com/shangxiaofei/p/10251214.html)

