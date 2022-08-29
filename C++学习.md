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

四条规则：[C++11特性：decltype关键字 - melonstreet - 博客园 (cnblogs.com)](https://www.cnblogs.com/QG-whz/p/4952980.html)

C++中的枚举enum：

一般枚举类型有如下三个缺点：

* 作用域不受限（全局），不同枚举变量成员名称相同时无法通过编译
* 枚举类型会隐式转换为int（？）
* 枚举类型变量实际类型无法确定，无法支持枚举类型前向声明。

对于第一个问题：

C语言可以在枚举变量成员前加上统一前缀来解决。

C++：1.使用命名空间namespace

​			2.使用类或结构体

对于后面两个问题，C++引入enum class：(本身作用域不再是全局的)

```c++
enum class color:int		//这里int指明enum存储的实际类型
{
    RED = 0,
    BLUE
};
//实际在访问枚举类的成员值时，必须用如下形式进行转换：
int temp = static_cast<int>(color::RED);
```

#### extern "C"

背景：C++支持函数重载，但C不支持（编译后生成的编译符号不同）。因而在C++函数调用C库函数时，因为符号的不同，编译时会提示找不到相应符号。

解决方法：使用extern "C"来解决。

注意：extern "C"是C++的语法，一般放在cpp文件或者cpp文件的头文件中。

对于C++文件调用C库函数：

如有a.c，a.h，main.cpp。在main.cpp中调用a.h头文件：

```c++
extern "C"{
    #include "a.h"
}
```

对于C文件调用C库函数：

如有a.cpp，a.h，main.c:

```c++
//对于a.h
#ifndef A_H
#define A_H
extern "C"{
    int func(int,int);
}
#endif
//对于main.c还要用extern声明要调用的函数：
extern int func(int,int);
```

另外一种通用的方法：

```c++
//在头文件中使用宏定义：
#ifdef __cplusplus
extern "C"{
#endif
int func(int,int);
#ifdef __cplusplus
}
#endif
```

疑问：

测试中使用gcc，g++编译cpp与c文件生成链接文件，编译结果（用nm -C查看.o链接文件）：

g++：		C文件：支持重载		cpp文件：支持重载

gcc：		 C文件：不支持重载    cpp文件：支持重载

为什么不直接用g++编译C库文件，可以避免extern "C"的使用？

#### 友元函数/友元类

友元函数：普通函数访问某个类的私有或保护成员。

友元类：一个类的成员函数访问另一个类的私有或保护成员。

友元关系不可传递（B是A的友元类，C是B的友元类，但C不能访问A的私有变量或保护变量）

友元关系不可继承（B是A的友元类，C继承A，B不可访问C的私有变量或保护变量）

友元关系单向性（指的是友元类？）

友元函数：

```c++
friend <类型><友元函数名>(<参数表>)
```

友元函数实现在类的外部，友元函数形式需要在类内进行声明。

[CPlusPlusThings/basic_content/friend at master · Light-City/CPlusPlusThings (github.com)](https://github.com/Light-City/CPlusPlusThings/tree/master/basic_content/friend)

友元类：

```c++
friend class <类名>
```

#### 函数指针：

````c++
//格式如下：
void (*pointer_func)(int);                              //1)
//具体含义：定义了一个指向函数体格式为void func(int)的指针（参数列表为void，返回值为int的函数）
//具体函数
void func_test(int aa);								//2)
//区别于：
int *func(int);
//func本身是函数地址，此为一个函数入参为int，返回值是int *的函数声明
````

用途：（以1）与2）为例，令pointer_func=func_test）

1.函数调用。调用格式：pointer_func(1)或者（*pointer_func）(1)

2.传参。pointer_func(1)或者（*pointer_func）(1)本身值为调用函数后的返回值

注意：对函数指针的赋值需要对应返回值与参数列表，否则编译出错。

#### 内联inline

一种用空间换时间的提高代码效率的方式。

具体实现是，内联的函数在编译时编译器在内联函数调用处就把函数代码进行替换，从而省去了在实际运行时进行函数调用的开销。

如何定义内联函数？

1.类内就进行定义的函数为隐式的内联函数。

2.类内声明，在实际定义时在函数最前面加上inline关键字。

3.普通函数：先声明后定义，定义时最前方加上inline关键字。

需要注意：

1.内联函数适用于实际函数是1-10行长度的代码。并且不能包含递归，循环等结构体，否则既损失空间也损失时间。

2.虚函数也可以进行内联，是否进行内联是在代码编译过程中编译器决定的。当虚函数在实际调用时呈现多态性质，那么编译器不会对该调用进行内联，只有编译器能够确定调用的内联函数实际属于哪个类才会进行内联。

#### 宏

对应宏定义中的“#”，常常跟在宏中的参数前，此符号会将参数字符串化。（只能用以有传入参数的宏定义，必须将#置于宏定义体中的参数名前）

对于传入参数有空格的情况：

1.传入参数前后的空格会被省略。2.传入参数名间的空格会转换成一个空格大小并将二者连接

对于宏定义中的“##”，分隔连接符，将宏定义的多个形参转换成一个实际参数名。

1.##进行形参连接，##前后的空格可有可无

2.##连接后的参数名需要有实际的意义（有预先的定义）

3.##后的参数本身也是一个宏，##会阻止宏的展开（？）

对于宏定义的“\”，标识换行符，但需注意“\”前需要有空格。

由于内核架构限制用到空宏导致出现warning，避免warning：

```c++
#define EMPTYMICRO do{}while(0)
```
