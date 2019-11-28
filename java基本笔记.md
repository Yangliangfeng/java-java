### Java基本比较

* 基本概念
```
1. JVM, JRE, JDK的概念
  1) JVM（java Virtual Machine） java虚拟机
  
  2）JRE （java runtime environment）java运行环境（jvm + java程序运行的核心类库）
  
  3) JDK （java development kit） java开发工具包（java开发工具+jre）
  
2. JVM, JRE, JDK之间的关系
   
   JDK = JRE + 开发工具集
   JRE = JVM + Java SE标准类库
```
* java的命名规范
```
1. 包名：多单词组成时所有字母都小写： xxyyzz

2. 类名，接口名：多个单词组成时，所有单词的首字母大写：XxYyZz

3. 变量名，方法名：多个单词组成时，第一个单词首字母小写，第二个单词开始首字母大写：xxxYyyZzz

4. 常量名：所有字母都大写。多个单词时，每个单词用下划线连接：XXX_YYY_ZZZ
```
* java的数据类型
```
1. 基本数据类型
    1）数值类型：
        （1）整数（byte（1字节=8位， -128-127）, short（2字节）, int（4字节）, long（8字节））
        （2）浮点（float（4字节，有效数字精确7位）, dobule（8字节））
    
    2) 字符类型：char（1字符 = 2字节）
    
    3）布尔类型：boolean

2. 引用数据类型
    1）类：class
    
    2）接口：interface
    
    3）数组
```
* 位移的操作
```
1. 左位移 相当于直接乘以 *2^n，其中n是位移的位数

2. 右位移 相当于直接除以 /2^n 取整，其中n是位移的位数

```
