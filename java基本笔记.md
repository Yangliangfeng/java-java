### Java基本知识

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
* 逻辑运算操作
```
1. 位操作
   1） 左位移 相当于直接乘以 *2^n，其中n是位移的位数  5 << 1

   2） 右位移 相当于直接除以 /2^n 取整，其中n是位移的位数 5 >> 1

2. 逻辑与运算“&”
   1&1 = 1  1 & 0 = 0  0 & 1 = 0 0 & 0 = 0
   与逻辑“&&”之间的区别是：当符号左边的是false时，&继续执行符号右边的运算。&&不再执行符号右边的运算
   
3. 逻辑异或 
    1^0 = 1  1 ^ 1 = 0  0 ^ 0 = 0

4. a++与++a之间的区别
   b = a++ 表示先把a的值付给b，然后，a再加1，所以， b = 1
   b = ++a 表示先把a的值加1，然后再给b，所以 b = 2
   
```
* 数组元素
```
1. 数组的初始化值
   1）数组元素是整型的：0
   2）数组元素是浮点型：0.0
   3）数组元素是char型的：0 或者 '\u0000',而非'0'
   4) 数组元素是引用数据类型： null
```
* JVM内存结构---类的解析
```
编译完程序以后，生成一个或者多个字节码文件，我们使用JVM中类的加载器

和解释器对生成的字节码文件进行解释运行。意味着，需要将字节码文件对应的

类加载到内存中，涉及到内存的解析。

虚拟机栈，即为平时提到的栈结构。我们将“局部变量”存储在栈结构中。

堆，我们平时将new出来的结构（比如：数组，对象）加载在堆空间中。

补充：对象的属性（非static的）加载在堆空间中。

方法区：类的加载信息，常量池，静态域
```
* 面向对象
```
1. 方法重载与重写：
    1）重载
    在一个类中，可以允许有一个以上的同名方法，但是，参数的个数或者参数的类型不一致
    
    （跟方法的权限[public, protected, private]，返回值类型，参数的变量名无关）
    
    2）重写
      子类继承父类以后，可以对父类中同名同参数的方法（非静态方法static），进行覆盖操作
      （1）子类重写的方法的权限修饰符不小于父类重写的方法修饰符
      （2）特殊情况，子类不能重写父类中声明为private的方法
      （3）子类重写方法的返回值类型要么相同，要么是其子类
      （4）子类重写方法抛出的异常类型不大于被重写的方法抛出的异常类型
      
    3）静态绑定和动态绑定
      （1）对于重载而言，在方法调用之前，编译器就知道了所要调用的具体方法，这称为“静态绑定”
      
      （2）对于多态，只有等到方法调用的那一刻，编译器才会知道所要调用的具体方法，这称为“动态绑定”
      
 2. Java的传值机制
     java是按照值传递的方式：
     1）如果参数是基本的数据类型，此时，实参赋给形参的是实参真实存储的数据值
     2）如果参数是引用数据类型，此时，实参赋给形参的是实参存储的数据地址值
     经典案例：
     public static void main(String[] args) {
        MyIndex.first();
    }
    public static  void first() {
        int i = 5;
        Value v = new Value();
        v.i = 25;
        second(v, i);//考察java的传递方式是值传递
        System.out.println(v.i);
    }
    public  static  void second(Value v, int i) {
        i = 0;
        v.i = 20;
        Value val = new Value();
        v = val;
        System.out.println(v.i + "  "+i);
    }
    class Value{
      int i = 15;
    }
    //结果是： 15  0
              20
```
* java四种访问权限修饰符
```
修饰符            类内部         同一个包         不同包的子类              同一个工程
private            yes            
(缺省)             yes             yes
protected          yes             yes              yes
public             yes             yes              yes                    yes

说明：
  对于class的权限修饰只可以用public和default(缺省)
  1）public 类可以在任意地方访问
  2）default类只可以被同一个包内部访问
  3）构造器的权限跟类的权限是一致的
```
* JavaBean
```
1. JavaBean 是一种java语言写成的可重用组件
2. 所谓JavaBean,是指符合如下标准的Java类：
   1）类是公共的
   2）有一个无参的公共构造器
   3）有属性，且对应的有get，set方法
```
* this关键字
```
1. this可以用来修饰，调用：
    属性， 方法，构造器

2. 一般情况下，调用属性和方法时，可以省略。特殊情况下，如果构造器或者方法的形参和类的属性同名时，
    我们就必须显式的使用“this.属性名”来表明调用的是变量属性，而非形参。
    
3. this调用构造器
   1）在类的构造器中，可以通过“this(形参列表)”方式，调用本类中其他的构造器
   2）构造器中不能通过“this(形参列表)”方式来调用自己
   3）规定：“this(形参列表)”必须放在调用构造器的行首
   4）构造器中最多只能声明一个“this(形参列表)”，来调用其他构造器
```
* extends继承
```
子类继承父类所有的方法和属性（包括私有方法和属性），只是由于父类的封装性

子类不能够直接调用

java只支持单继承
```
* super
```
super调用构造器
  1）必须声明在子构造器的行首
  2）在构造器的行首没有显式的声明“this（参数列表）”或者“super(参数列表)”，则默认调用是父类的
     空参构造器：super()
  3) 在类的构造器中，至少有一个类的构造器使用了“super(参数列表)”，调用父类的构造器
```
* 多态
```
1. 可以理解为一种事物的多种形态

2.对象的多态
  父类的引用指向子类的对象（或子类的对象赋给父类的引用）
  
3. 多态的使用：虚拟方法调用
   编译时，只能调用父类声明的方法；运行时，实际调用的是子类重写的方法
   总结：编译看左边，运行看右边
   
4. 多态使用的前提：
    1）类的继承关系
    2）方法的重写

5. 对象的多态性，只适用于方法，不适应于属性

6. 有了对象的多态性以后，内存中实际上市加载了子类持有的属性和方法，但是由于变量声明为父类类型，
   
   导致编译时，只能调用父类声明的属性和方法。子类持有的属性和方法不能调用。
   
7. 如何才能调用子类持有的属性和方法
   向下转型：使用强制类型转换符
   
8. 对于引用数据类型，比较的是两个引用数据变量的地址是否相同

9. 多态的实例变量的调用，都是来自的父类的，编译运行都看左边

10. 多态的经典例题
    public class ExecTest {

    public static void main(String[] args) {
        Base1 base = new Sub1();

        base.add(1,2,3);

        Sub1 sub = (Sub1)base;
        sub.add(1, 2, 3);

    }
  }

  class Base1 {
      public void add(int a, int... arr) {
          System.out.println("base1");
      }
  }

  class Sub1 extends Base1 {
      //默认数组和可变参数类型是相同的
      public void add(int a, int[] arr){
          System.out.println("sub_1");
      }
      public void add(int a, int b, int c) {
          System.out.println("sub_2");
      }
  }
   
```
* “==”和equal的区别
```
1. “==” 是运算符
   （1）可以使用在基本数据类型变量和引用数据类型变量中
   （2）如果比较的是基本数据类型变量，比较的是两个变量保存的数据值是否相等
        如果比较的是引用数据类型变量，比较的是两个对象的地址是否相同，即两个引用是否指向同一个实体
        
2. equals()方法使用
   （1）是一个方法，而非运算符
   （2）只能适用于引用数据类型
   （3）说明：在Object类中定义的equals()和==作用是相同的，比较两个对象的地址是否相同
   （4）像String,Date,FIle，包装类等都重写了Object类中的equals()方法，重写以后，比较的不是两个引用地址是否
        相同，而是比较两个对象的“实体内容”是否相同
```
* 基本数据类型，包装类，String类之间的转化
```
1. 基本数据类型   =>   包装类
   1）通过构造器：Integer i = new Integer(1);
   2) 自动装箱(java5.0之后的特性)： Integer i = 12;
   
2. 包装类    =>    基本数据类型
   1）调用包装类的方法：int a  = i.intValue()  ---其中i是包装类的对象，而intValue是包装类的方法
   2）自动拆箱（java5.0之后的特性）： int a = i;
   
3. 基本数据类型，包装类    =>  String
   1) 连接运算： String str1 = num + "";
   2) 调用String重载的valueOf(xxxx)方法： String fl = String.valueOf(12.3f);
   
4. String类型   =>  基本数据类型，包装类
   1）调用包装类中的parseXxx方法: int a = Integer.parseInt(str1);
```
* static关键字
```
1. 使用static修饰方法：静态方法
    1）随着类的加载而加载，可以通过“类.静态方法”的方式进行调用
    2）静态的方法中，只能调用静态的方法或者属性
       非静态的方法中，既可以调用非静态的方法或者属性，也可以调用静态的方法或者属性
    3）在静态的方法中，不能使用this关键字，super关键字
    
2. 使用static修饰属性：静态变量
    1）静态变量随着类的加载而加载，可以通过“类.静态变量”的方式进行调用
    2）静态变量的加载早于对象的创建
    3）由于类只会加载一次，则静态变量在内存中也只能存一份：存在方法区的静态域中。
```
* 代码块
```
1. 代码块的作用：用来初始化类和对象
2. 代码块如果有修饰的话，只能是static
3. 分类：静态代码块 VS 非静态代码块
4. 静态代码块
   1）内部可以有输出
   2）随着类的加载而执行，而且只能执行一次
   3）作用：初始化类的信息
   4）如果一个类中定义了多个静态代码块，则按照声明的先后顺序执行
   5）静态代码块的执行要优于非静态代码块的执行
   6）静态代码块内只能调用静态的属性，静态的方法，不能调用非静态的结构
   
5. 非静态代码块
    1）内部可以有输出语句
    2）随着对象的实例化而执行
    3）每实例化一个对象，就执行一次非静态代码块
    4）作用：可以在实例化对象时，对对象的属性进行初始化
    5）如果一个类中定义了多个非静态的代码块，则按照声明的先后顺序执行
    6）非静态的代码块可以调用静态的属性和方法，也可以调用非静态的属性和方法

6. 赋值的顺序
   1）默认初始化
   2）显式的赋值 或者 代码块
   3）构造器
   4）通过方法来设置
   
7. 代码块的执行顺序
   由父及子，静态先行
```
* static 和 final修饰的对象
```
1. static用来修饰： 属性， 方法 代码块

2. final用来修饰： 类， 属性， 方法
```
* abstract抽象类和方法
```
1. 抽象类中一定有构造器，便于子类实例化调用
```
* interface接口(java8以后的特性)
```
1. 接口中定义的静态方法，只能通过接口来调用

2. 通过实现类的对象，可以调用接口中的默认方法
   如果实现类中重写了接口中的默认方法，调用时，仍然调用的是重写以后的方法
   
3. 如果子类（实现类）继承的父类和实现的接口中声明了同名同参数的默认方法
   那么子类在没有重写次方法的情况下，调用的是父类的同名同参数的方法

4. 如果实现类实现了多个接口，而这多个接口定义了同名同参数的默认方法，
   那么，在实现类没有重写次方法的情况下，报错-----> 接口冲突
   这就要求我们必须在实现类中重写次方法
   
```
* 成员内部类
```
1. 内部类分类;
   1）成员内部类（静态，非静态）
   2）局部内部类（方法内，代码块内，构造器内）
```
* throw和throws的区别
```
1. throw 抛出一个异常的对象，生成异常对象的过程。声明在方法体内

2. throws属于异常处理的一种方式，声明在方法的声明处
```
* 进程   线程
```
1. 进程
    正在运行的一段程序。是一段动态的过程：有它自身的产生，存在和消亡的过程 ------生命周期
    所以，进程是动态的。进程作为资源分配的单位，系统在运行时为每个进程分配不同的内存区域
    
 2. 线程
    是一个程序内部执行的一条执行的路径
    线程作为调度和执行的单位，每个线程拥有独立的运行栈和程序计数器（pc），线程切换的开销比较小
    一个进程的多个线程共享相同的方法区和堆
    
3. 一个java程序至少有3个线程
  1）main（）主线程
  2）gc（）垃圾回收线程
  3）异常处理线程
```
* java创建线程的二种方式
```
1. 继承Thread类，创建的步骤如下：
   1）继承Thread类
   2）重写run方法
   3）实例化一个对象
   4）实例化的对象调用start()方法
   
2. 实现Runnable接口
   1）创建一个实现了Runnable接口的类
   2）实现类实现Runnable接口中的run()方法
   3）创建实现类的对象
   4）将此对象作为参数传递给Thread类的构造器中，创建Thread类的对象
   5）通过Thread类的对象调用start()方法
   
3. 比较两种创建线程方法的异同：
   开发中：优先选择：实现Runnable接口的方式
   1）实现的方式没有类的单继承性的限制
   2）实现的方式更加适合处理多个线程共享数据的情况
   
   3）两种方式都重写了run方法，将线程要执行的逻辑声明在run()方法中
```
* 线程的五种状态

![](https://github.com/Yangliangfeng/java-java/raw/master/Images/thread.png)


