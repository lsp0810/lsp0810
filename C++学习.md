# C++学习

## 类&对象

类用以指定对象形式，包含数据表示法和用于处理数据的方法。

类中数据和方法称为类的成员，函数在一个类中称为类的成员。

类没有定义任何数据，定义了类的含义，即类的对象包括什么以及在这个对象可以执行的操作。

```C++
class classname	// 关键字 类名
{
    Access specifiers:		//访问修饰符  private/public/protected
    	Data members/variables;	//变量
    	Member functions(){}	//方法
    
};			//注意分号
```

类提供对象蓝图，对象是根据类来创建。如定义类Box

```c++
class Box{
    public：
        double a;
    	double b;
    	double c;	//上述三个成员变量
    	double get_tiji();
};
#定义类函数
double Box::get_tiji(){
    return a*b*c;
}
```

实例化两个对象

```c++
Box box1;
Box box2;
```

两个对象都有各自的成员变量，通关'.'来进行访问（似有与受保护成员除外）。

## 类成员函数

类成员函数是指那些把定义与原型写在类定义内部函数的函数。为类的一个成员，可以操作类的任意对象，也可以对象中所有成员。



位域：

区别“联合“与”结构“。

使用C语言实现C++中类的继承与多态

思路：

* 用struct结构体代替class类，用函数指针代替类内成员属性或方法。
* 用结构体嵌套实现类的继承
* 多态-->函数指针的不同。

#### 类型修饰符const：

作用：

* 定义常量
* 类型检查：1.与define相区别，一个可以安全检查一个不可以；2.const定义的变量只有整数与枚举
* 对程序有保护作用
* 节省空间，减少内存分配

const对象默认为文件局部变量（默认不带extern），非const对象默认为extern对象。

const与指针：

```c++
char *a;		//最初
const char *a;			1
char const *a;			2
char * const a;			3
```

关键：看const与*位置。一种是修饰数据类型（1，2），一种是修饰指针本身（3）。

指针与const对象：

const对象只能由void const*指针指向，且const对象的值不能通过对象本身以及指针来进行更改。

非const对象可以由普通指针指向（如int变量可以由int*指针指向，且此时对象的值可以由对象本身或者对象指针进行更换），也可以由const指针指向（如int可以由int const *指向，但此时对象的值只能由对象本身进行改变，而不能由指针）。

const本身字段应用于参数返回值以及函数入参。

```c++
//一种特殊情况：当函数入参为引用时：
void func(A &a);
//相较于void func(A a),不用再函数内部产生参数的临时复制，因而效率较高，但会改变参数a内容
//可以用const解决
void func(const A a);
//对于常见的数据类型，如int double float等，不需要采用const ***&，因为此类数据类型不存在
//构造与析构过程，采用引用与单纯的值复制效率相当
```

类内的const：

const成员变量----初始化列表初始化（构造函数后）。

const函数（常函数）才可以操作常量以及常对象（？怎么个操作法？）---1

const对象只能访问const成员函数，非const对象可以访问任意成员函数。---2

```txt
我的理解：
1是说明在一个对象内部对一个成员函数，只有成员函数是const类型才可以访问其他const函数，否则无法访问；2是说明对于对象本身，若是const修饰的对象，那么对象本身是不能够访问非const函数成员的
```

对于类内的const变量，除去列表初始化，也可以：

```c++
//第一种
//类内定义
static const int a;
//类外初始化
const int A::a=0;

//第二种,仅针对C++11
//定义时直接初始化
static const int a=0;
//或者
const int a=0;

//此外对于static变量的初始化
//先类中进行声明
static int a;
//类实现文件初始化
int A::a=0;
//C++11对于单纯static变量没办法声明并初始化（个人理解）
```

#### decltype使用

对表达式类型进行查询，但不进行计算。

```c++
decltype(expression)
```

作用：

* 推导表达式类型
* 与using与typedef复用，用以定义类型
* 重用匿名类型
* 结合auto，追踪函数返回值类型

```c++
template <typename T>
auto multiply(T x,T y)->decltype(x*y)
{
    return x*y;
}
```

左值，右值，将亡值：（C++11之后）

[(28条消息) C++11 左值、右值、右值引用详解_hyman_yx的博客-CSDN博客_c++11 左值和右值](https://blog.csdn.net/hyman_yx/article/details/52044632?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-3-52044632-blog-108396338.pc_relevant_multi_platform_whitelistv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-3-52044632-blog-108396338.pc_relevant_multi_platform_whitelistv4&utm_relevant_index=4)

decltype规则：

