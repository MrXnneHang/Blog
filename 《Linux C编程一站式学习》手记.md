# 《Linux C编程一站式学习》阅读手记。

## 这不是认认真真的笔记。

我接触编程到现在也2年了，学校开的水课组成原理，C++，数据结构与算法，操作系统也都上过了。

所以，这里不再是一种系统性的学习，而是希望在之前的程度上再深挖一点，比如学校课真的水（操作系统把linux的基本命令和系统管理过一遍就完了，其他都是考概念，一两天突击一下就过去了。）导致我自己虽然学过这些课，但我完全没有底气说自己真的懂这些。

我会找出我在意的地方记录，以及，记录是主观的。

**后来我再看，我发现我写得实在是，太水了，而且又太浅了。不足以给别人看，但，毕竟是我第一次记录，也可以看看后来能改进多少。**




## Compile型和Interpret型的高级语言区别。

Compile型代表:C语言

main.c  : source code,遵循C语言语法的文本文件。

---- compiled  经过gcc的编译。

> **常见标准库在glibc中，并且glibc还提供一些linux的系统底层接口。**

->main.out: 可执行的文件，通常是可以被系统接受的机器码。

```bash
gcc main.c -o main.out
./main.out
```

Interpret型代表:Python

不同于C,有这样的语法结构:

```python
>>a=1
>>b=2
>>c=a+b
>>c
3
```

它有记忆性，可以逐行运行。

另外，它和操作系统似乎是有剥离的，是一种虚拟平台。

和C相比，C编译后是可以直接被操作系统接受的，是直接运行在操作系统内核的，不知道这么讲是不是准确。

而python通常需要一个python的解释器，它运行时是一种类似虚拟环境的东西。有一层软隔离吧，虽然说必然得是操作系统运行的，但我觉得因为python的pyrc也不是机器码，而是得由python解释器像上面那样逐行运行，因此我认为它和操作系统的无关性和隔离性要比C语言更好。

但这样会有的缺点就是一样的指令运行会更慢。比如循环打印，自增自减等等，应该也是由于虚拟化。

### 高级语言的移植性：

C语言相对于汇编和机器语言算得上高级语言。

因为我记得汇编和机器语言一一对应。

而C语言不管mac上还是windows上还是linux上写的语法都一样，区别只在编译器，最后编译时翻译出来的机器码和调用的底层不同。所以在不同的机器之间移植不需要变动源代码而只需要更换编译器，这就很好的能被迁移到不同的系统平台。只要有人写编译器吧我觉得。

而Python那边，也类似，它用的解释器被miniconda给封装起来了，我虽然linux和windows都有用，但是没仔细观察python解释器是不是不同。

但至少这也很简单的说明了为什么linux和windows之间的环境不能直接copy。

首先python底层有C，C在两个系统之间编译必然是不同的。那么python环境也是不能移植的，有点牵强好像。

>##### 1.1 什么是 CPython？
>
>CPython 是 Python 语言的一个主要实现，它是用 C 语言编写的 Python 解释器。当你在大多数 Linux、Windows 或 macOS 上安装并运行 Python 时，你实际运行的就是 CPython。Python 代码在 CPython 中会被解析、编译成字节码（bytecode），然后由解释器逐行执行这些字节码。
>
>因此，Python 和 C 之间的直接关系就是：Python 的核心解释器 CPython 是由 C 语言实现的。
>
>##### 1.2 CPython 的工作流程
>
>- **解析器（Parser）**：CPython 将 Python 源代码解析为语法树（AST）。
>- **编译器（Compiler）**：将 AST 编译为 Python 字节码，这是一种中间形式，用于虚拟机解释执行。
>- **Python 虚拟机（PVM, Python Virtual Machine）**：解释并执行这些字节码。PVM 本身也是用 C 语言编写的。
>
>这意味着，当你编写并执行 Python 代码时，C 语言代码在后台负责解释和执行你编写的 Python 代码。

这么看起来，python是C的宝宝，从解释到运行全都在C的一个虚拟化环境下。这个虚拟化环境和系统应该是一个主机和docker关系。

区别主要在于，是否直接翻译成操作系统内核接受的机器码并且直接由操作系统内核来运行。



## printf和scanf:

因为学的时候是c++，printf用得少，scanf没见过。

C++咋写来着

```c++
std::cout<<"hello world";

std::cin<<"your age"<<age;
```

完全忘了cin这东西。

scanf和cin类似，也和python的input()差不多。

咱之后不写C++了，学好C就可以了。C++语法规则太他妈多了，而且类也不好写。要面向对象用python它不好吗？

> ps:C++ 里用的是iostream,C用的是stdio。





## C语言内存管理布局和变量，地址，指针存储分布：



示例：

```c
#include<stdio.h>

void cal(){
        int i;
        printf("%d\n",i);
        i = 777;
        printf("%d\n",i);
}

int main(){
        cal();
        cal();

        printf("=======================\n");
        cal();
        printf("Hello\n");
        cal();
        return 0;
}
```

```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# ./main
0
777
777
777
=======================
22091
777
Hello
22091
777
```

第二次执行时，22091发生了变化

```cmd
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# ./main
0
777
777
777
=======================
22073
777
Hello
22073
777
```



关注点在于i的未初始化值的打印，

四次分别是:

```cmd
0
777
22091
22091
```

0的那次，是函数初始化。

777，是第一次初始化777后。

22091/22073,是执行了printf后，虽然printf的内容不一样。【**以及，为什么两次不一样**】

对于**未初始化的全局变量和静态变量**，会被统一初始化为0。

![Snipaste_2024-09-06_09-01-32](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409060903980.jpeg)

这样不会导致值溢出，因为静态变量和全局变量的数据类型一开始就定义好，包括数组也会有固定长度。所以全部为0之后修改也不会溢出。

而我们的i作为局部变量，它的0不是全局统一初始化的，而是掐好分配到地址上面的变量就是0，可能是栈在程序启动的时候也做了一次类似格式化的操作，把地址上的值都格成0了。

### 局部变量的位置：栈上，执行时分配，结束时释放。几十MB。

### 函数指针（地址）的位置：栈上，包含返回地址（该示例中主函数地址）

栈的起始点是高地址，先进后出。

非常符合嵌套函数执行。

比如a -> b -> c -> d。

但是在返回时是依次返回的，是d->c->b->a

比如说简单写一个:

```python
i = 10
def a():
   i+=1
   b()
   print(i)
def b():
    i+=1
    c()
    print(i)
def c():
    i+=1
    d()
    print(i)
def d():
    i+=1
    print(i)
```

这一定是个垃圾程序，我都懒得想这到底打印出来会是什么样的。

但是你可以发现，print(i)的顺序是,d->c->b->a.

也就是说，每次我一次次进入函数，但是退出也是一个个退出的。

先进，后出，a最先进来，但是是最后出来的。

函数指针地址放在栈上是非常合适的。



#### 我们可以解释为什么两次"./main"执行时打印出的垃圾值都不一样

22091/22073,虽然很接近。

连续执行都差不多在21800~22200之间。

这个可能是主函数的地址，或者执行函数的地址。

我个人偏向于主函数地址。

因为两个垃圾值是一样的，而两次printf的地址必然是不一样的，所以，这个很大概率是main函数的地址的一部分。告知printf的返回是main地址。



### 动态内存和堆空间：

new,malloc,realloc可以分配堆内存，是随时可以更改的，只要剩余空间充足，那都可以作为堆空间。

### 代码段和数据段：

data segment:全局变量，静态变量位置。也有一个特殊区块是未初始化的数据段。

text segment:指令存放位置。之前提到，函数的地址在栈上，而text segment是指令，也就是函数的执行内容。而且指令应该有冗余，统一放一起可以节省空间。

都是预分配固定大小的，应该在编译的时候就计算好了。或者说在程序执行的时候就计算好了。

按理来说，机器指令不应该有内存管理意识，所以应该还是归功于C语言的编译器。



### 程序指针pc:

不在内存里，在处理器里，应该和那个高速缓存有关，英文名称我忘了。



### 内存布局图。

![Snipaste_2024-09-06_08-53-00](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409060933802.jpeg)

segment预分配，固定大小，程序运行中一般不变。

Stack固定大小，反复初始化释放。

Heap，除非free或者delete，否则一直存在，并且可以一直动态扩展，直到顶到了Stack。

### 哲学问题:

#### 哲学问题a:但通常会有大聪明在函数里面放一个int a[30000],这不爆炸吗？这个应该怎么处理。

会爆炸，考虑转全局，或者动态分配到栈，只在函数域里留下一个指针。

#### 哲学问题b：如果全局声明指针然后用动态分配内存那么它应该在哪里？

指针存在data segement里，这依然不会改变大小。

而后续分配在堆内存里。



不仅感叹天才，非常简单的设计，但是非常的高效率。  


## 一直以来让我误会的递归:

```c
#include<stdio.h>

int factorial(int n)
{
    if (n == 0) {
        printf("x1");  // 添加了分号
        return 1;
    } else {
        int jiecen = factorial(n - 1);
        int result = jiecen * n;
        printf("x%d", n);  // 格式化打印n值
        return result;
    }
}

int main()
{
    int n = 4;
    int res = factorial(n);
    printf("\n%d!=%d\n", n, res);  // 修正了 "prinf" 为 "printf"
    return 0;
}
```

```bash
gcc recursion.c -o recursion
./recursion
```

```cmd
x1x1x2x3x4
4! = 24
```

我之前一直以为输出应该像这样:<br>
```cmd
x4x3x2x1
4! = 24
```
但错了两点，一个是递归里，每次都会先看到recursion(n-1),先进到函数里，和我上次的a->b->c->d的逻辑一样,进到最后，最先的打印的是最后调用的，然后return是一个栈的逻辑。<br>
第二个是，递归会有一个base case,这里是0! = 1,最后一次调用的是factorial(0),它被乘到了1!的后面。所以会有x1x1。<br>

### 递归练习:Euclid求最大公约数
我觉得我先写成python再写成C会更好一点。
约束：
```python
if a%b==o:
      b is the gcd of a and b.
else:
    gcd = gcd(b,a%b)
```

```python
def base(a,b):
    if a%b==0:
        return b
    else:
        return base(b,a%b)
def gcd(x,y):
  if x>y:
        return base(x,y)
  else:
        return base(y,x)
        
if __name__ == "__main__":
    print(gcd(12,8))
```

然后让gpt翻译了一个c<br>
```c
#include <stdio.h>

// 定义 base 函数，递归计算最大公约数
int base(int a, int b) {
    if (a % b == 0) {
        return b;
    } else {
        return base(b, a % b);
    }
}

// 定义 gcd 函数，用于调用 base 函数来计算最大公约数
int gcd(int x, int y) {
    if (x > y) {
        return base(x, y);
    } else {
        return base(y, x);
    }
}

int main() {
    int x = 12;
    int y = 8;
    // 调用 gcd 函数并打印结果
    printf("GCD of %d and %d is: %d\n", x, y, gcd(x, y));
    return 0;
}
```
c语言写起来太抓狂，要造轮子尽量让gpt来造，这边懂个原理就好了。<br>
```cmd
GCD of 12 and 8 is: 4
```
不过这个例子里只有一次递归。12%8 = 4 -> 8%4 = 0 -> 4 <br>
而且这个递归是顺序执行的，因为主要计算都在条件判断里。条件判断在调用递归之前。<br>
另外对于12，它的return顺序依然是,4->4->4,一个base case里的4经过两次递送给了最初。依然是从后向前。<br>
**不管外部计算顺序在递归前还是调用递归后。对于递归本身，它总是从base case开始向前传播的。** <br>  


## 编译和链接:makefile,cmake

当我同时写了gcd.c和math.cd，大概像这样。

`math.c`<br>
```C
// 定义 base 函数，递归计算最大公约数
int base(int a, int b) {
    if (a % b == 0) {
        return b;
    } else {
        return base(b, a % b);
    }
}

// 定义 gcd 函数，用于调用 base 函数来计算最大公约数
int gcd(int x, int y) {
    if (x > y) {
        return base(x, y);
    } else {
        return base(y, x);
    }
}
~         
```
`gcd.c`<br>
```C
#include <stdio.h>

int base(int a, int b);

int gcd(int x, int y);

int main() {
    int x = 12;
    int y = 8;
    // 调用 gcd 函数并打印结果
    printf("GCD of %d and %d is: %d\n", x, y, gcd(x, y));
    return 0;
}
```

然后试图编译时发生了错误:<br>
```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# gcc gcd.c
/usr/bin/ld: /tmp/ccBAK3aM.o: in function `main':
gcd.c:(.text+0x25): undefined reference to `gcd'
collect2: error: ld returned 1 exit status
```
以前只用visual studio还真不知道需要手动建立连接。<br>

gpt给了我几种选择:<br>
![img_1.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409201131910.png)
![img.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409201131871.png)

不管哪一种，都是很麻烦的。而且如果多次编译岂不是要心态爆炸。<br>

然后就引出来了，makefile。我以前连makefile用来做啥的都不知道（捂脸）。<br>

Cmake也干的差不多的事情，但是选择困难，只好都稍微认识一下。 <br>

#### makefile 示例:

创建`./makefile`<br>

```makefile
# 定义编译器和编译选项
CC = gcc
CFLAGS = -g -Wall

# 定义源文件列表
SRCS = gcd.c math.c

# 定义目标文件列表 (将所有 .c 文件转换为 .o 文件)
OBJS = $(SRCS:.c=.o)

# 定义最终生成的可执行文件
TARGET = gcd_program

# 默认规则 (make 的默认目标是编译可执行文件)
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $(TARGET) $(OBJS)

# 定义 .o 文件生成规则，自动从 .c 文件编译成 .o 文件
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 清理编译生成的文件
clean:
	rm -f $(OBJS) $(TARGET)
```
可以自定义SRCS,TARGET,clean。其他感觉可以保持原样。不过这个clean都可以自己写，真的很自由，这个大概就是删掉生成的.o和最终文件。<br>

另外，make file似乎和C kernel style时一种编码风格，都是用tab来缩进的。 <br>

**编译：**<br>
```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# make clean
rm -f gcd.o math.o gcd_program
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# ls
breakpoints.txt  gcd.c  makefile  math.c
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# make
gcc -g -Wall -c gcd.c -o gcd.o
gcc -g -Wall -c math.c -o math.o
gcc -g -Wall -o gcd_program gcd.o math.o
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# ls
breakpoints.txt  gcd.c  gcd.o  gcd_program  makefile  math.c  math.o
```

#### CMake 示例:

创建`CMakeLists.txt`: <br>
```cmake
cmake_minimum_required(VERSION 3.10)

# 定义项目名称
project(MyProject)

# 指定 C 标准
set(CMAKE_C_STANDARD 99)

# 添加可执行文件
add_executable(gcd_program gcd.c math.c)
```
Cmake原理就是自动化生成makefile，并且自动处理依赖。<br>

**编译：**<br>
```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# vim CMakeLists.txt
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# mkdir build
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# cd build
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing/build# cmake ..
```
前面我们已经大概推理出来makefile比cmake底层，那么cmake大概率需要手动安装。<br>

但就在我安装完后编译，依然出现了以下错误:<br>

```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing/build# cmake ..
-- The CXX compiler identification is unknown
CMake Error at CMakeLists.txt:4 (project):
  No CMAKE_CXX_COMPILER could be found.

  Tell CMake where to find the compiler by setting either the environment
  variable "CXX" or the CMake cache entry CMAKE_CXX_COMPILER to the full path
  to the compiler, or to the compiler name if it is in the PATH.


-- Configuring incomplete, errors occurred!
See also "/home/xnne/Linux_C_Programing/build/CMakeFiles/CMakeOutput.log".
See also "/home/xnne/Linux_C_Programing/build/CMakeFiles/CMakeError.log".
```
一眼没看懂。大概是编译器找不到。好像是找不到c++的编译器。<br>

**解决方案：**<br>

![img_2.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409201131400.png)

![img_3.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409201131333.png)

区别在于`project()`新增了一个C。<br>

我采用了第二种。<br>

随后编译成功:<br>
```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing/build# vim ../CMakeLists.txt
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing/build# cmake ..
-- Configuring done
-- Generating done
-- Build files have been written to: /home/xnne/Linux_C_Programing/build
```
生成了一个两百多行的Makefile,而我们当时只用了十几行。<br>

确实更严谨，而且更安全，但多了一层封装，没有那么接地气了。<br>

最后大概结论大概这样：如果是从0开始构建的个人项目，只是做开源，那么makefile足够，而且有种掌控感。但如果要团队合作或者考虑到频繁传播和多平台传播，这边建立一个CmakeList给其他用户会更专业一些。<br>

终于认识了makefile和cmake。gcc,g++也懂了=-=。万恶的Visual Studio，封装程度越高，也就意味着越低的操作性。这种操作性的缺乏在编程的时候有时候会化为一种无力感，特别是当碰到一个底层出了bug但我什么都做不了。有一种"空中楼阁"的感觉。时不时温习一下底层可以摆脱这种自我怀疑。 <br>

## 调试：gdb

有意思的是，在问gpt的时候不小心打成了gdc，然后就得知了有一个神似C语言但是可以自动管理内存的D语言。 <br>

编程语言千千万，原理懂C就够了。 <br>

这里只学了点基本操作，比如设置断点，单步执行，查看变量值。 <br>

依然借用上次的代码:<br>

`math.c`<br>
```C
// 定义 base 函数，递归计算最大公约数
int base(int a, int b) {
    if (a % b == 0) {
        return b;
    } else {
        return base(b, a % b);
    }
}

// 定义 gcd 函数，用于调用 base 函数来计算最大公约数
int gcd(int x, int y) {
    if (x > y) {
        return base(x, y);
    } else {
        return base(y, x);
    }
}
~         
```
`gcd.c`<br>
```C
#include <stdio.h>

int base(int a, int b);

int gcd(int x, int y);

int main() {
    int x = 12;
    int y = 8;
    // 调用 gcd 函数并打印结果
    printf("GCD of %d and %d is: %d\n", x, y, gcd(x, y));
    return 0;
}
```
这样是不是显得我水字数？但直接copy过来就不用索引到上面看了。<br>

编译这边直接用makefile。<br>

使用gdb之前需要加入-g选项。<br>

对于单文件编译是这样的:<br>

```bash
gcc -g gcd.c -o gcd
```

这里我们用makefile是在编译选项里加入:<br>

```makefile
# 定义编译器和编译选项
CC = gcc
CFLAGS = -g -Wall
```
### 编译:<br>
```bash
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# make
gcc -g -Wall -c gcd.c -o gcd.o
gcc -g -Wall -c math.c -o math.o
gcc -g -Wall -o gcd_program gcd.o math.o
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# ls
CMakeLists.txt  breakpoints.txt  gcd.c  gcd.o  gcd_program  makefile  math.c  math.o
root@DESKTOP-3I1GRP0:/home/xnne/Linux_C_Programing# gdb ./gcd_program
```

### 查看函数源码:<br>
```bash
(gdb) list base //查看base函数源码。
1       // 定义 base 函数，递归计算最大公约数
2       int base(int a, int b) {
3           if (a % b == 0) {
4               return b;
5           } else {
6               return base(b, a % b);
7           }
8       }
9
10      // 定义 gcd 函数，用于调用 base 函数来计算最大公约数

(gdb) list gcd
6               return base(b, a % b);
7           }
8       }
9
10      // 定义 gcd 函数，用于调用 base 函数来计算最大公约数
11      int gcd(int x, int y) {
12          if (x > y) {
13              return base(x, y);
14          } else {
15              return base(y, x);
```
### 按照文件查看源码:<br>
```bash
(gdb) list gcd.c:1
1       #include <stdio.h>
2
3       int base(int a, int b);
4
5       int gcd(int x, int y);
6
7       int main() {
8           int x = 12;
9           int y = 8;
10          // 调用 gcd 函数并打印结果
(gdb)
11          printf("GCD of %d and %d is: %d\n", x, y, gcd(x, y));
12          return 0;
13      }
14
15
```
它每次都会列出10行，然后再列出10行，如果没列完你可以直接点回车接着列出。空格的意思似乎是重复执行上一步<br>

### 设置断点:<br>
```bash
(gdb) list gcd
6               return base(b, a % b);
7           }
8       }
9
10      // 定义 gcd 函数，用于调用 base 函数来计算最大公约数
11      int gcd(int x, int y) {
12          if (x > y) {
13              return base(x, y);
14          } else {
15              return base(y, x);

(gdb) break gcd:12
Breakpoint 1 at 0x11e2: file math.c, line 12.
```
我习惯直接对函数设置断点。也可以把gcd改成math.c对文件设置断点。<br>

### 保存和加载断点：
保存:
```bash
(gdb) save breakpoints breakpoints.txt
```
加载:
```bash
(gdb) source breakpoints.txt
```

### 运行:<br>

这里不过多讲怎么调试，因为这个需要实战，等下次你发现你真的需要调试了，但脑子不够用，硬看看不出来，就用gdb来调试吧。还有一些查看变量值等等功能。你就之后问gpt吧。 <br>

### 退出:<br>
```bash
(gdb) quit
```
防止你不会退出。(●ˇ∀ˇ●)<br>


## 结构体：

### 定义，初始化：

python写得太久，我都忘了C语言的结构体是怎么定义的。 <br>

```c
struct student {
    char name[20];
    int age;
    float score;
};

int main() {
    struct student stu1 = {"Tom", 18, 90.5};
```

python里的=什么都能做，但是C里的=只能初始化。 <br>

定义需要始终struct关键字。另外，初始化也需要struct关键字。 <br>

python那边是:
```python
class student:
    def __init__(self,name,age,score):
        self.name = name
        self.age = age
        self.score = score

stu1 = student("Tom",18,90.5)
```

如果不想初始化，也可以:<br>
```c
stu1 = {0};#全部初始化为0
```
通常情况下，不建议。<br>

并不需要带class关键字。 <br>

### 嵌套结构体：

```c
struct Segment {
        struct complex_struct start;
        struct complex_struct end;
 };
```

c里面似乎没有类继承的概念，但嵌套也差不多。<br>

## 字符串:

C语言似乎只用char的字符数组来表示字符串。<br>

然后按照之前C++的经验，"Hello World"这个应该是一个const char*。<br>

C里面没有string这个东西，真的是万分感谢。之前用C++的时候，因为string不上不下的，又不能像python那样自由地索引和更改，和char和char*,const char*混用还存在类型不能强制转换和不能直接赋值的问题。<br>

## 代码风格:

### 缩进:

尝试全部使用tab缩进，除非文档规定。<br>

### 注释:

尝试多使用: <br>
```c
/*
*
*这是一个多行注释
*
*/
```
少的使用:<br>
```c
//这是一个单行注释
//这是一个单行注释
```

函数等等和python的类似。 <br>

宏定义也需要注释。<br>

### 命名:

* 可以缩写。<br>
* 拒绝匈牙利命名法。「用变量名的前缀记录变量的类型，例如iCnt、pMsg、lpszBlk」<br>
* 全局变量和全局函数命名尽量长，明确用途。<br>
* 不要使用中文拼音。<br>

### 函数:

* 命名和python一致。<br>
* 尽量短【不超过48行】，只实现一个功能。如果需要很长，考虑分割多个函数。<br>

### 规格化工具：

ident,astyle等等。 <br>

正好有素材的时候试试。<br>



