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

