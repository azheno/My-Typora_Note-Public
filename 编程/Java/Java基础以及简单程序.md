# Java基础以及简单程序



java程序中，源代码程序后缀为`.java`



## JDK和JRE

### JDK

JVM：java虚拟机，真正运行Java的设备

核心类库：`System.out.println("hello")`，java自身提供的内置的函数 

开发工具： java，javac，jdb，jhat等命令用来操作java程序 

由JVM，核心类库，开发工具组成的一个可以编写java集成工具包，被称为JDK 



### JRE

JVM：java虚拟机，真正运行Java的设备

核心类库：`System.out.println("hello")`，java自身提供的内置的函数 

运行工具： java等命令用来运行java程序 

由JVM，核心类库，运行工具组成的一个可以运行java程序的一个集成环境，被称为JRE 



### JDK、JRE、JVM的关系 

JDK作为开发工具包含了JRE和JVM

JVM作为运行Java的设备，是每个运行和编写java不可缺少的一部分





## 注释

```java
// 注释信息 
/* 注释信息 */ 
/** 文档注释 **/
```



## 关键字： 



### class

用于创建或者定义一个类，类是Java基本的组成单元

```java
public class hello {
    
}
```



## 字面量

数据在程序中的书写格式



### 字面量的分类

| 字面量类型 | 说明                           | 举例                     |
| ---------- | ------------------------------ | ------------------------ |
| 整数类型   | 不带小数点的数字               | 666，-88                 |
| 小数类型   | 带小数点的数字                 | 13.14，12.44             |
| 字符串类型 | 用双引号引起来的内容           | “hello world”,"你好世界" |
| 字符类型   | 用单引号引起来的，内容只有一个 | ‘A’,'i','你'             |
| 布尔类型   | 表示真假                       | 只有两个值，true，false  |
| 空类型     | 表示空值                       | 表示方法为null           |



### 特殊字面量

#### \t

制表符，在打印的时候，将前面的字符长度补到8，或者补到8的倍数，最少补一个空格，最多补8个空格

```java
package com.test01;

public class hello {
    public static void main(String[] args) {
    System.out.println("alice" + '\t' + "man");
    System.out.println("tom" + '\t' + "women");
    }
}
//结果： 
alice   man 
tom     women
```







## 变量



### 概念：

指程序运行期间可以发生变化的量  

### 作用：

变量是一个存储数据的容器，即存储单元，他的功能就是用来存放程序中需要处理的数据

### 变量的基本操作： 

*   赋值 
*   取值

### 变量的定义格式

```java
数据类型 变量名 = 数据值;
int a = 1 
```





## IDEA



### 项目结构： 

*   project（项目） 
    *   module（模块）
        *   package（包）
            *   class（类）



### 第一个脚本

```java
package com.test01;

public class hello {
    //叫做main方法，表示程序的主入口
    public static void main(String[] args) {
    //输出语句，用于打印语句信息
    System.out.println("hello");
    }
}
```



