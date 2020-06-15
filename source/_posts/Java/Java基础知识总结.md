---
title: Java基础知识总结
cover: true
top: true
date: 2019-09-14 17:05:29
group: java
permalink: java-basic-summary
categories: Java后端
tags:
	- Java
	- 知识总结
keywords:
	- Java常见基础知识总结
summary: 本文总结了Java常见的、易混淆的基础知识，常用工具等。
---


### 1\. substring()方法

`String`类的`substring()`方法在JDK6与JDK7的区别:

-    **JDK6**：截取的`String`对象会共享字符数组 (对于从很长的字符串中截取很小的字符串时，可能会引起内存泄漏，因为内部使用的字符数组不能被回收）
-    **JDK7**：截取的`String`对象会创建新的字符数组

### 2\. Integer缓存机制

-    当整数在-128到127范围时，会使用缓存的`Integer`对象，而不会新建对象。
-    缓存机制只有在自动装箱场景下有用。如：`Integer i = 10`
-    `Byte`, `Short`, `Long`, `Character`都有类似缓存机制。`Character`适用范围为`0`到`127`.

### 3\. 浮点型

### 4\. SynchronizedList与Vector区别

-   `SynchronizedList`可以将所有`List`的子类转成线程安全类
-   `SynchronizedList`遍历时需要手动进行同步，`Vector`的遍历方法加了同步锁
-   `SynchronizedList`可以指定锁定的对象

### 5\. 常见垃圾回收器

-   **串行回收器 （-XX:+UseSerialGC）**

    Young：

    - 串行

    - 复制算法： 1个`Eden`区，2个`Survivor`区

    Old：

    - 串行

    - 标记压缩算法

-   **并行回收器 (-XX:+UseParallelGC)**

    Young:

    - 多CPU并行

    - 复制算法 （与串行相同）

    Old：与串行回收器相同

-   **并行压缩回收器（-XX:+UseParallelOldGC)**

    Young: 与并行回收器相同

    Old：并行

-   **并发标识清理回收器（CMS）-XX:+UseConcMarkSweepGC**

    Young: 与并行回收器相同

    Old：并发

### 6\. 类型自动提升

当表达式中包含多个基本类型时，会发生自动提升

-   `byte`，`short`和`char`会被提升为`int`
-   提升到与表达式中最高等级操作数同样的类型：
    `byte` -> `short`
    `char` -> `int` -> `long` -> `float` -> `double`

### 7\. 隐式类型转换

复合赋值运算符（如`+=`， `-=`， `*=`）会进行隐式类型转换。

- 会自动将计算结果值强制类型转换为左侧变量的类型

    e.g,:
    ```java
    short i = 5;
    i += 2; //不等于 i = i + 2, 会报编译错误
    ```

### 8\. 类型擦除

当把有泛型类型的对象赋值给一个没有泛型类型的变量（原始类型），泛型类型信息将被擦除。

### 9\. switch语句

- `switch`语句只支持：`byte`, `char`, `short`, `int`, `Character`, `Byte`, `Short`, `Integer`, `String`, `enum`

- 在`switch`语句中使用枚举类时，`case`分支中访问枚举值时不能使用枚举类名作为限定，而要直接使用枚举值。

### 11\. if/else语句

`if/else`语句使用的基本规则：总是优先把包含范围小的条件放在前面

### 12\. 循环语句

如果在for, while或do循环中不使用花括号时，第一个语句不能是局部变量声明语句

### 13\. instanceof

-   如果引用变量为`null`，则永远返回`false`
    ```java
    String s = null;
    boolean result = s instanceof String  // 返回false
    ```

-   编译时判断编译类型，运行时才判断引用变量引用的对象实际类型

### 14\. 单例模式

单例模式的类一般需要实现一个私有的`readResolve()`方法，防止反序列化时产生新的对象。

### 15\. 方法重载

-   调用方法时传入的实际参数会被向上转型为方法的形参类型
-   编译时匹配类型更精确的方法进行调用，即传入参数与形参类型更接近
-   如果多个参数匹配结果产生冲突，则无法通过编译
-   被重载的方法必须改变参数列表，其他修改是optional的

### 16\. 方法重写

-   参数列表必须完全与被重写方法的相同。
-   返回类型必须是被重写方法的相同类型或派生类型（子类，实现类）
-   访问权限不能比被重写方法低
-   子类必须具有访问父类方法的权限，才能重写父类方法
-   `final`方法不能被重写

### 17\. 非静态内部类

-   编译器会默认为内部类的构造器添加外部类作为第一个参数
-   不能有静态成员变量

### 18\. static

-   只能用于修饰类内部成员：Field，方法，内部类，初始化块，内部枚举类
-   静态方法由变量的声明类型调用
-   静态内部类无法访问外部类的非静态成员

### 19\. 异常

-   `catch`捕获多个异常时，应先捕获子类异常，再捕获父类异常，否则会报编译错误
-   可以在程序任何地方catch 任何运行时异常（`RuntimeException`及其子类异常），或者异常超类`Exception`
-   `catch`只能捕获`try`块内程序可能抛出的非运行时异常或其父类异常
-   子类重写父类方法时，不能声明抛出比父类方法类型更多，范围更大的异常

### 20\. 性能测试工具

-   LoadRunner
-   JMeter

### 21\. 自动化测试工具

-   WinRunner
-   QTP
-   Selenium