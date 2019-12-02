### Java常见的面试题

* 考察知识点---包装类(Wrapper)
```
1. 题目如下
    Object o1 = true ? new Integer(1) : new Double(2.0);
    System.out.println(o1)    //1.0 
    分析：三元运算符两边的数据类型必须一致，因此int类型自动提升为double类型，所以，最后输出1.0
    
2. 题目如下：
   Integer i = new Integer(1);
   Integer j = new Integeer(1);
   System.out.println( i == j)   //false  ----"=="地址赋值
   
   Integer m = 1;
   Integer n = 1;
   System.out.println(m == n); // true
   
   Integer x = 128;
   Integer y = 128;
   System.out.println(x == y); //false
   
   //分析：Integer内部定义了IntegerCache结构，IntegerCache中定义了Integer[],保存了从-128-127范围内的整数。
           如果我们使用自动装箱操作的方式，给Integer赋值的范围在-128-127范围内时，可以直接使用数组中的元素
           不用再去new了；第三个小问是因为x和y都超出了128，所以，必须重新new这个整型的对象
```
