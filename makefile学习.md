取自：[跟我一起写Makefile (PDF重制版)](https://seisman.github.io/how-to-write-makefile/Makefile.pdf)

C/C++代码生成可执行文件过程：

.c/.cpp文件 ----> 中间代码文件.obj windows/.o UNIX ----> 可执行文件

第一步是编译compile，第二步是链接link

1. 第一步需要代码语法正确，函数与变量声明正确。正确编译出目标文件需要告诉编译器头文件位置以及头文件与源文件语法正确。一般一个源文件对应一个中间目标文件。（对于未被声明的函数或变量，编译器还是会生成中间文件，但会报出警告，在实际link链接时会因为找不到函数原型而报错）

第二步需要链接函数与全局变量。主要指明相应的中间文件进行链接。但很多时候，中间文件过多造成不便。便有给这些中间文件打包的文件，windows是Library File(库文件)，.lib文件，UNXI是Archive File，也即是.a文件。

### Makefile文件介绍

以C工程为例，makefile编写完后，使用make命令需要达到：

* 若工程没被编译过，那么所有C文件需要被编译链接
* 若某几个C文件被修改，那么只编译被修改的C文件并链接
* 若某几个头文件被修改，那么需要重新编译引用被修改头文件的源文件，并进行链接。

```makefile
target ... :prerequisites ...
	command
	...
```

target：目标文件、执行文件、标签（伪目标）

prerequisites：生成target需要依赖的文件和/或target

command：该target要执行的命令（任意shell指令）

makefile规则：

prerequisites中若有一项文件比target要新，command就会被执行。

在写好一个文件的依赖关系后，需要写要执行的指令，需要换行+tab

make的工作过程（输入make后会执行的操作）：

1. 寻找当前目录下名字为Makefile或makefile的文件
2. 找到后找到文件中第一个目标文件，将此文件作为最终的目标文件
3. 如果最终目标文件不存在，或者是目标文件依赖项文件比最终目标文件新，那么就会执行定义的命令生成最终目标文件。
4. 如果最终目标目标文件依赖项不存在，那么make会在makefile中找到该依赖项的依赖关系，先生成此文件再执行最终目标文件生成指令

上述根据最终目标依赖关系来间接生成其他依赖文件的过程，决定了类似于makefile中make clean此类命令不会被执行,这是由于clean此类标签不包含在最终可执行文件的依赖关系中,因而不会被自动执行,若是想执行可以输入make clean之类的指令,而非make

#### makefile使用变量

makefile中的变量可以理解为C语言的宏

变量以字符串来表示,例如:

```makefile
object = main.o file1.o file2.o
#如何访问?
target:$(object)
	cc -o target $(object) 
```

对于中间目标文件进行删除,增添只需修改object变量即可

#### makefile的自动推导(make的隐晦规则)

自动推导依赖文件:如whatever.o,其会自动将whatever.c添加到依赖文件中;自动推导指令:以上述为例,其会自动推导:cc -c whatever.c

.PHONY:作用:声明目标为伪目标文件

另一种风格的makefile:[跟我一起写Makefile (PDF重制版)](https://seisman.github.io/how-to-write-makefile/Makefile.pdf) 中第七页的makefile为例.

makefile可以以头文件为分类基准,对中间文件进行分类.

#### 更为保险的make clean

```makefile
#一般
clean:
	rm target $(obj)
#更为保险的做法
.PHONY:clean
clean:
	-rm  target $(obj)
```

rm前的"-"表示某些文件可能会出现问题,但忽略并继续做后续的事情

clean一般放在makefile的最后

Makefile包含五类内容:

* 显式规则:显式规则用以说明如何生成一个/多个目标文件;显式规则包含要生成的目标文件,生成目标文件的依赖以及如何生成的命令.命令开头必须以Tab键开始
* 隐晦规则:指的是makefile文件本身支持自动推导的那部分
* 定义变量:字符串,类似于C语言的宏
* 文件的指示:
  * 类似于C中的include,引用另一个Makefile
  * 指定Makefile文件中有效部分,类似于C中的预编译
  * 定义一个多行的命令
* 注释:以"#"开头,可以用"\\#"进行转义

编译makefile指定文件:

```shell
make -f/--file MakefileName
```

#### makefile引用其他makefile

语法形式:

```makefile
include filename
```

几点注意事项:

* include前可以有字符,但不能是tab键开始
* 一个include可以包含多个filename,不同filename用空格隔开
* filename可以用makefile内的变量,也可以用省略形式的文件说明,例如:a.mk,*.mk,$(mk_file)都是可以的
* make执行makefile时,先找寻include保含的makefile,并在include位置将包含的makefile内容安置在此
* makefile找寻include包含文件的路径规则如下(在include的文件不指明绝对路径或相对路径的条件下):
  * 先在当前目录下寻找
  * make执行时指定的-I/--include-dir
  * 目录<prefix>/include下寻找(一般:/usr/local/bin或/usr/include)

忽略include的报错(好像是仍然会执行)

```makefile
-include filename
#与下面等效
sinclude filename
```

GNU的make执行过程:

1)读入所有makefile

2)读入被include的其它makefile

3)初始化文件的变量

4)推导隐晦规则,分析所有规则

5)为所有目标文件创建依赖关系链

6)根据依赖关系决定哪些目标需要重新生成

7)执行生成命令



### 如何书写

规则:

```makefile
targets:prerequisites
	command
	...
#或:
targets:prerequisites;command
	command
```

注:targets可以是多个目标,以空格隔开即可

##### make中的通配符

* ' ~ ':UNIX下:~/test表示当前用户$HOME目录下的test目录;~usrname/test表示用户usrname的宿主目录下的test目录;windows/MS-DOS下,用户无宿主目录,~所指目录根据环境变量"HOME指定".
  * 宿主目录:
* ' * ':代替一系列文件,如*.c代表当前目录下所有.c文件

```makefile
#对于一个目录下的a.o与b.o用变量包含
object = *.o		#object值即为*.o,并不会自动展开
object := $(wildcard *.o)#里面变量展开

$(patsubst %.c,%.o,$(wildcard *.c))
```

* ' ? ':' $? '表示一个自动化变量

make寻找依赖文件以及目标文件路径:

* 当前目录(最高优先级)
* VPATH = PATH1:PATH2:PATH3:...,优先级按照顺序依次降低

大写VPATH是makefile的一个特殊变量,小写vpath是make的关键字,vpath使用方法:

```makefile
#为符合模式pattern的文件指定搜索目录<directories>
vpath <pattern> <directories>
#清除符合模式<pattern>的文件搜索目录
vpath <pattern>
#清除所有已被设置好的文件搜索目录
vpath
```

<pattern>可以用%进行指定:

```makefile
vpath %.h ../path1
#表示makefile中所有.h文件在当前目录搜索不到后都去../path1去搜索
```

可以连续使用vpath指定路径寻找顺序,也可以用":"进行分隔

```makefile
vpath %.c path1
vpath %   path2
vpath %.c path3
#上述搜索顺序:当前目录-->path1--->path2--->path3
vpath %.c path1:path3
vpath %   path2
#上述搜索顺序:当前目录-->path1--->path3--->path2
```

#### 伪目标

伪目标的指令需要执行需要显示的指明(make+标签名)

标签名不能和已存在文件同名(可以通过.PHONY来显式指明)

使用伪目标可以一下产生多个可执行文件

```makefile
all:prog1 prog2
.PHONY:all

prog1: a.o b.o
	command1

prog2: c.o d.o
	command2
```

#### 多目标：

使用自动化变量实现，例如$@(表示目标的集合)

在makefile中$表示执行一个函数

#### 静态模式：

基本语法形式如下：

```makefile
<targets ...> : <target-pattern> : <prereq-patterns>
	<commands>
	...
```

这种模式下，<targets>是目标的一个集合（保含的文件数目大于等于目标数目），<target-pattern>是对目标集合的筛选，指明目标的模式，<prereq-patterns>是对目标集合的二次筛选，结果是第一次筛选目标的依赖项，下面为具体示例：

```makefile
objects = foo.o bar.o other.c

all:$(objects)
.PHONY=all

$(objects):%.o:%.c				-----1
	$(CC) -c $(CFLAGS) $< -o $@
#$<为自动化变量，表示第一个依赖文件
#$@为自动化变量，表示目标集
#故而上述makefile等效于：
foo.o:foo.c
	$(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o:bar.c
	$(CC) -c $(CFLAGS) bar.c -o bar.o
```

上述1语句，从$(objects)中取.o文件作为目标文件，从过滤出的.o文件中用.c代替.o获得源文件。

```makefile
files = file1.o file2.o file3.elc

$(files):%.o:%.c					-----2
	$(CC) -c $(CFLAGS) $< -o $@
$(fileter %.o,$(files)):%.o:%.c		-----3
	$(CC) -c $(CFLAGS) $< -o $@
$(fileter %.elc,$(files)):%.elc:%el
	emacs -f batch-byte-compile $<
```

2与3有啥区别？

#### Makefile自动生成目标文件的依赖性

背景：对于大型的工程，有时可能会删除或增添头文件，这样生成目标文件的依赖都要进行更改，给工程维护带来不便。C/C++编译器提供关键字选项来查看依赖：

```shell
cc/gcc/g++ -M source.c/source.cpp
#上述命令会把标准库的头文件也包含进来
cc/gcc/g++ -MM source.c/source.cpp
#上述命令会剔除标准库头文件的依赖
```

如何将-MM生成的字符串应用到Makefile？可以将编译器通过-MM指令生成的字符串重定向输出到.d文件中（为makefile文件）。转换成功后便可以直接include到我们自己的makefile中去，具体.d文件生成指令：

```makefile
%.d:%.c
	@set -e;rm -f $@;\
	$(CC) -MM $(CPPFLAGS) $< > $@.$$$$;\
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ >$@;\
	rm -f $@.$$$$
```

上述规则最终完成的任务是（以source.c为例）：

1.生成source.c的依赖规则，保存到source.d.xxxx文件中

2.将source.o:source.c替换为source.o，source.d：source.c并将其保存到source.d中

3.删除临时文件source.d.xxxx

对于上述语句解释：

@关键字表示make不输出改行命令

set -e表示后面命令出错时立即退出

上述四行命令以‘；’隔开，' \ ‘进行换行连接,目的是能够使得前面的命令作用于后面的命令，这里是set -e起作用。

第二行表示用编译器将源文件依赖输出并写入$@.$$$$文件中

第三行表示将$@.$$$$进行批量替换，并将替换后的结果写入$@中

第四行表示删除临时文件

因而实际要make的makefile中只需将.d文件include进去即可。

### 书写命令

#### 显示命令

@添加到命令行前，make不会将其显示

```shell
make -n/--just-print
#对命令只显示但不执行
make -s/--silent/--quiet
#全面禁止命令的显示
```

#### 命令执行

前一条命令作用于后一条命令

#### 命令出错

添加’-‘来忽略命令出错/或者是用make -i/--ignore-errors来全局声明。

另外有make -k/--keep-going表示命令出错，终止该规则但继续执行其他规则。

？：make-k跟-i有啥本质区别么

#### 嵌套执行make

#### 定义命令包

形式如下：

```makefile
define name
xxxx
xxxx
endef
```

使用命令包$(name)即可

### 使用变量

变量的命名可以有字符、数字、下划线（可以数字开头）

不可有：’、‘，’#‘、’=‘、空格、回车

变量大小写敏感

变量需初始化，使用时需要用$开头解析，并给变量套上小括号()或者是大括号{}，使用$字符需要用$$表示

变量----替代

变量的值可以用其他变量来定义，其中“其他变量”不用事先定义好，可以在Makefile其他位置出现。

上述变量定义都是使用“=”操作符，这样会产生坏处，即变量的递归调用，例如：

```makefile
A = ${B}
B = ${A}
```

这种调用方式make会报错。

其次在变量中使用函数会使得make编译缓慢，并且会产生未知错误。

解决的方法：“:=”操作符

这种操作符前面的变量不能使用后面的变量

```makefile
x := foo
y := $(x) bar
x := later
#等价于
y := foo bar
x := later

y := $(x) bar
x := foo
#等价于
y := bar
x := foo
```

如何用:?来把变量定义为空格？

```makefile
#way1
empty:=
space:=$(empty) #
#empty变量什么也不包含，space变量哪一行中#前有多少空格space就被定义了几个空格
#way2
empty:=
space:=$(empty) $(empty)
#empty含义如上，其中第二行两个$(empty)有多少空格，space就被定义了几个空格
```

另外的赋值符号?=(条件赋值)

```makefile
FOO ?=bar
#等价于
ifeq ($(origin FOO),undefined)
	FOO = bar
endif
```

含义：若FOO未被定义过那么FOO的值为bar，否则保持不变

变量值替换：

格式：

```makefile
$(var:a=b)
${var:a=b}
```

含义：将var字符串中以a结尾的字符替换为b

这里结尾是指找到a字符后，其后面必须跟空格或者是结束符才可以进行替换。

例子：

```makefile
foo:=a.o b.o c.o
bar:=$(foo:.o=.c)
#或者
foo:=a.o b.o c.o
bar:=$(foo:%.o=%.c)
#第二种方法用%说明foo里的pattern都是以.o结尾的同样的作用
```

将变量的值当作变量

变量追加赋值：+=

```makefile
var := value1 value2
var := $(var) value3
#等效于
var := value1 value2
var += value3

var = value1 value2
var = $(var) value3
#等效于
var = value1 value2
var += value3
```

+=赋值情况：若之前变量未定义或者是前面变量定义为’=‘，那么+=赋值与=类似（但所涉及的递归调用问题会被make自动解决，不用担心）；若上次变量定义为’:=‘，那么+=与:=类似。

#### override指示符

变量是由make命令行参数设置，而在makefile中对此变量的赋值被忽略。此类变量赋值：

```makefile
override <variable>; = <value>

override <variable>; := <value>

override <variable>; += <value>
```

对于多行变量定义，可以在define前加上override指示符

#### 多行变量

与命令包相同的命令

文章中解释多行变量定义中不以tab键开头，make不会将其认为是命令，但命令包中同样也没以tab键开头啊

#### 环境变量

make开始运行时系统环境变量可以载入Makefile文件中

若是Makefile中已定义此变量或是make命令行带入，系统环境变量值被覆盖；make命令行中加上’-e‘，系统环境变量覆盖Makefile中定义的变量。

#### 如何使用Makefile中的局部变量？

语法：

```makefile
<target...>:<variable-assignment>
<target...>:override <variable-assignment>
```

其中<target...>指明了变量作用的目标范围，变量值在目标所引发的一系列规则有效，且变量若是和全局变量名称冲突，这里定义的局部变量优先级更高；其次override是针对make命令行代入的变量或者系统环境变量。

定义符合模式的变量作用范围

语法：

```makefile
<pattern ...> : <variable-assignment>
<pattern ...> : override <variable-assignment>
#原文<pattern ...>后加上了分号
```

override作用同上

```makefile
%.o : CFLAGS = -O
```

上述语句作用是将CFLAGS变量作用到所有以.o为结尾的目标上

### 条件判断

条件判断：比较变量值或者是比较变量与常量的值

示例，在书中第37页

```makefile
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

条件表达式的书写有以下形式：

```makefile
#其一
ifeq(<arg1>,<arg2>)
ifeq '<arg1>' '<arg2>'
...
#省略号表示第二种方式对于<arg1>与<arg2>其引号可以为单引号也可以为双引号
#其二
ifneq (<arg1>,<arg2>)
ifneq '<arg1>' '<arg2>'
...
#省略号表示第二种方式对于<arg1>与<arg2>其引号可以为单引号也可以为双引号
#其三
ifdef <variable-name>
#通过判断变量是否为空来判断表达式是否为真
#其四
ifndef <variable-name>
#与其三含义相反
```

makefile的条件表达式中，<conditional-directive>以及else与endif都是允许空格与#注释符，但不允许tab打头、

其次判断条件中不要以自动化变量进行判断（make在读取makefile时就把条件表达式计算完毕并根据条件表达式选择了相应的语句，而自：W'q动化变量是在makefile运行时才推导出来的）

make不允许将条件语句分成两部分放到不同文件中

### 函数调用

函数调用用$来标识，格式：

```makefile
$(<function> <argument>)
#或者
${<function> <arguments>}
```

函数名与参数间以空格相分隔，参数内部以’,‘相分隔

函数1：subst

```makefile
$(subst <from>,<to>,<text>)
```

作用将text字符串中的from字段替换为to字段

```makefile
$(subst ee,EE,feet on the street)
#输出
fEEt on the strEEt
```

函数2：patsubst

```makefile
$(pathsubst <pattern>,<replacement>,<text>)
```

<text>以单词进行分割（用空格、TAB、回车、换行为界限），检查每一个单词是否符合<pattern>，若符合对其进行替换，替换内容为<replacement>

函数3：strip

```makefile
$(strip <string>)
```

去除<string>里开头与结尾的空字符

函数4：findstring

```makefile
$(findstring <find>,<in>)
```

查找<in>中的<find>字符串，找到返回<find>,否则返回空字符串

函数5：filter

```makefile
$(filter <pattern...>,<text>)
```

返回<text>中符合<pattern>的字符串（可以包含多个pattern）

函数6：filter-out

```makefile
$(filter-out <pattern...>,<text>)
```

反过滤函数，返回不符合pattern的字符串（可以有多个pattern）

函数7：sort

```makefile
$(sort <list>)
```

对<list>里的单词以首字母为基准进行升序排列，最后返回排序后的字符串（sort会自动去除list中重复的单词）

函数8：word

```makefile
$(word <n>,<text>)
```

取<text>中第n个单词并返回

* n从1开始计数
* n大于text中单词数量时返回空字符串

函数9：wordlist

```makefile
$(wordlist <ss>,<e>,<text>)
```

函数返回text中从第ss个单词到第e个单词的字符串(正常情况)

若ss>text单词数，返回空字符串

若e>text单词数，返回从<ss>开始到<text>结束的字符串

若1<=ss<e<=text单词数，函数正常返回

ss>=e呢？

函数10：words

```makefile
$(words <text>)
```

返回单词个数

函数11：firstword

```makefile
$(firstword <text>)
```

返回<text>中第一个单词



***上述函数都是用来操作字符串的，下面是操作文件名***

函数12:dir

```makefile
$(dir <names...>)
```

取出文件名序列中第一个’/‘前的部分，没有’/‘的返回’./‘

问：可以返回文件绝对路径么？

函数13：notdir

```makefile
$(notdir <names...>)
```

从文件名序列中取出非目录部分（应该单单指文件名）

函数14：suffix

```makefile
$(suffix <names...>)
```

返回文件名序列的文件名后缀，没有后缀返回空字符串

函数15：basename

```makefile
 $(basename <names...>)
```

取文件名序列的文件名前缀部分。

```makefile
$(basename src/foo.c src-1.0/bar.c hacks) 
#返回值是 src/foo src-1.0/bar hacks
```

函数16：addsuffix

```makefile
$(addsuffix <suffix>,<names...>)
```

给文件名序列的每个文件名统一添加后缀

函数17：addprefix

```makefile
$(addprefix <prefix>,<names...>)
```

给文件名序列的每个文件名添加前缀

函数18：join

```makefile
$(join <list1>,<list2>)
```

list2的单词按顺序依次拼接到list1的单词上，最后返回<list1>

若：$(words,<list1>)>$(words,<list2>)，那么<list1>多出的部分保持不变;$(words,<list1><$(words,<list2>)，那么<list2>多出的部分按顺序粘贴到<list1>后

***以下为其余函数：***

函数19：foreach

```makefile
$(foreach <var>,<list>,<text>)
```

上述执行一个循环（直至<list>的单词被取完），以如下例子进行过程介绍：

```makefile
names := a b c d
files := $(foreach n,$(names),$(n).o)
```

$(names)=a b c d

foreach函数以单词为单位，每次从$(names)取一个单词作为依次循环，foreach将取得单词放入临时变量n中，临时变量再执行$(n).o的指令完成一个循环，并返回指令处理的结果（作为一个单词）。

因而最终改行命令一共进行四次循环，返回了四个单词，单词间以空格相隔，最终结果：a.o b.o c.o d.o

函数20：if函数

```makefile
$(if <condition>,<then-part>)
$(if <condition>,<then-part>,<else-part>)
```

注意：函数返回值为<then-part>或者是<else-part>计算结果，若是任一方需要运行但没有定义，那么返回空字符串

函数21：call函数

```makefile
$(call <expression>,<parm1>,<param2>,...,<paramn>)
```

以下是我对call函数的理解：<expression>可以被定义为一系列参数的组合表达式（用$(1)、$(2)等来表示参数顺序）。然后调用call函数后，表达式会将写$(1)、$(2)等的地方用实际参数表示，最后返回替换后的整个字符串。

此外call函数的参数中，从第二个参数开始若是有多余的空格，其会保留，因而保险而言，需要将多余空格去除。

函数22：origin函数

```makefile
$(origin <variable>)
```

作用：告知变量的来源，不推荐在<variable>使用$。器返回值如下：

* undefined：变量从来没有进行定义
* default：变量是默认定义，如CC
* environment：环境变量，-e参数没有被打开
* file：变量被定义在Makefile中
* command line：变量被命令行定义
* override：被override指示符重新定义
* automatic：自动化变量

函数23：shell

```makefile
$(shell <instruction>)
```

入参是shell指令，返回是shell指令的输出

#### 控制make的函数

error函数

```makefile
$(error <text ...>)
```

作用：产生致命错误，终止make运行，并提示<text...>消息

只有函数展开（被调用时才有作用）

包含“error”函数引用的命令被执行，定义中引用此函数的递归变量被展开。

warning函数

```makefile
$(warning <text...>)
```

与error函数类似，但自身不会产生致命错误使得make退出，只是提示相关信息，make继续

### make的运行
