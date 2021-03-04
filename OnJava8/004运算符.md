总的来说：
几乎所有的运算符只能操作基本类型，但是=、==和!=例外（几乎可以操作所有对象），String类支持+，和+=

1 加减乘除没什么好说的  
=  

2 赋值：（a=b）  
=
* b为基本类型：直接赋值给变量a  
* b为引用类型：把引用复制给a，指向同一个对象（将引用传递给方法的时候，仍然指向同一个对象）  

3 递增和递减  
=  
对于前递增和前递减（如 ++a 或 --a），会先执行递增/减运算，再返回值;  
而对于后递增和后递减（如 a++ 或 a--），会先返回值，再执行递增/减运算。  

4 关系运算符:产生一个布尔类型的结果  
=  
== 和 != 比较特殊，可以用于所有的基本类型（其他比较运算法不能用于boolean），而且可以用于测试对象的等价性。  
一些非常令人疑惑的问题就要向我们袭来，阅读下面的内容要保持清醒的头脑，阅读下面的内容要保持清醒的头脑，阅读下面的内容要保持清醒的头脑（重要的事情说三遍）：
1. ==和!=比较Integer
    ```
    // operators/Equivalence.java
    public class Equivalence {
        public static void main(String[] args) {
            Integer n1 = 47;
            Integer n2 = 47;
            System.out.println(n1 == n2);
            System.out.println(n1 != n2);
        }
    }

    输出结果：
    true
    false
    ```
    我们知道的是，==和!=比较的是引用，结果就该是false和true，但是和输出结果反了，这是因为Integer内部维护着一个IntegerCache的缓存，默认缓存范围是 [-128, 127]，没有像创建其他对象的时候那样创建两个对象。  
2. equal
    equal用来比较两个对象的内容是否相同。但是问题又出现了：
    ```
    // operators/EqualsMethod.java
    public class EqualsMethod {
        public static void main(String[] args) {
            Integer n1 = 47;
            Integer n2 = 47;
            System.out.println(n1.equals(n2));
        }
    }

    输出结果:
    true
    ```  
    毫无疑问。但是，但是，但是......
    ```
    // operators/EqualsMethod2.java
    // 默认的 equals() 方法没有比较内容
    class Value {
        int i;
    }

    public class EqualsMethod2 {
        public static void main(String[] args) {
            Value v1 = new Value();
            Value v2 = new Value();
            v1.i = v2.i = 100;
            System.out.println(v1.equals(v2));
        }
    }

    输出结果:
    false
    ```   
    WTF，对象内容不是一致的吗？  
    其实，是因为equals()的默认行为是比较对象的引用而非具体内容，除非在新的类中覆写equals()。

5 逻辑运算  
=
&& （AND）、||（OR）和 !（非）根据参数的逻辑关系生成布尔值 true 或 false  
逻辑运算中存在“短路”（整个表达式会在运算到可以明确结果时就停止并返回结果）  

6 字面值常量  
=  
对于Long类型的数值，结尾使用大写L或者l  
对于float类型的数值，结尾使用大写F或者f  
对于double类型的数值，结尾使用大写D或者d  
十六进制前面用0x或0X表示  
八进制前面用0表示  
二进制前面用0b或者0B表示  
可以在数字字面量中包含下划线 _，以使结果更清晰（仅限单 _，不能多条相连；数值开头和结尾不允许出现 _；F、D 和 L的前后禁止出现 _；二进制前导 b 和 十六进制 x 前后禁止出现 _）  

7 指数计数  
=

8 位运算：操作一个整型数字中的单个二进制位
=  
移位运算符：  
* 左移位运算符 << 能将其左边的运算对象向左移动右侧指定的位数（在低位补 0）  
* 右移位运算符有“正”、“负”值：若值为正，则在高位插入 0；若值为负，则在高位插入 1  
* “不分正负”的右移位运算符（>>>），无论正负，都在高位插入 0 （当无符号右移与赋值相结合时，若将其与 byte 或 short一起使用的话，则结果错误。取而代之的是，它们被提升为 int 型并右移，但在重新赋值时被截断，在这种情况下，结果为 -1） 

9 三元运算符  
=
`布尔表达式 ? 值 1 : 值 2`  
若表达式计算为 true，则返回结果 值 1 ；如果表达式的计算为 false，则返回结果 值 2。

10 字符串运算符：连接字符串  
=

11 类型转换：在适当的时候，Java 会将一种数据类型自动转换成另一种  
=
编译器会在必要时自动提升  
若将数据类型进行“向下转换”，可能会信息丢失（在执行“向下转换”时，必须注意数据的截断和舍入问题，从 float 和 double 转换为整数值时，小数位将被截断。）  


小知识：
1. Random  
```
// Create a seeded random number generator
//其中的47是种子（随机数生成器的初始化值，其始终为特定种子值产生相同的序列），每次启动程序，随机出来的结果是一致的
Random rand = new Random(47);
 int i, j, k;
// Choose value from 1 to 100:
//nextInt() 的参数设置生成的数字的上限，下限为零，为了避免零除的可能性，结果偏移1
j = rand.nextInt(100) + 1;
System.out.println("j : " + j);
k = rand.nextInt(100) + 1;
System.out.println("k : " + k);
```  

