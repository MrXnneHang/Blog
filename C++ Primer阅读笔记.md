# CPP Primer阅读笔记:

```cmd
这里都用CPP来指代C++，因为似乎连续两个加号会引起markdown的特殊语法。
```

我的所有代码都会放在:[https://github.com/MrXnneHang/CPP-Primer-Note](https://github.com/MrXnneHang/CPP-Primer-Note)<br>

## 1.1:开始

作者出题真的很刁钻，1.3嵌套注释以前还真不知道。四行代码把规则都概括了，很有水平，而且我喜欢这种方式，推论要自己得出来。<br>

所以我决定，把我当成初学者，把所有题都扫一遍。但并不是每一题都在这里记录，我记录的通常是额外的尝试或者拓展我之前认知的。<br>

### 1.1.1:编写一个简单的CPP程序

**选择编译方式**：<br>
在windows上面配置IDE让我放弃了。Visual Studio很大，而且太傻瓜式了。CLion要自己配置Cmake等工具链。<br>

所以这一次依然会是在wsl的ubuntu环境上运行。<br>

![img_2.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410140949207.png)

几乎没代码。注意的是，直接执行`./a.out`是看不到返回信息的。要用`echo $?`查看返回值。<br>

返回-1时，我的out时255。在python里-1通常是最后一个元素，这里应该是一个字节的最大值。<br>

但CPP异常退出的时候，通常是-65123456这种很长的，不止一个字节。<br>


### 1.2：初识输入输出(cin的缓冲区结构)

![img_3.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410140949512.png)

语法错误，把v1,v2后的`;`去掉就行。<br>

![img_6.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410141314379.png)

这里想要补充的是`std::cin`的缓冲区。<br>

我注意到有两种输入方式，一种是输入一行，用空格分隔。另一种是一行输入一个,然后回车，输入两次。<br>

你可以在这里看到我的测试缓冲区:[1.11.x.cpp](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/1.11.x.cpp)，我进行了多组测试，下面是推论。<br>

test1：<br>
推论:**cin传入的是第一个有效值，而不是被最后一个覆盖。**<br>
输入1 2 3 4 ,最终i值是1而不是4。<br>

test2:<br>
推论:**cin的缓冲区内是一个队列，如果一次性输入4个，但是只cin一次，那么会剩下三个存留在缓冲区内，在下一次cin>>时，不会再向用户键盘输入的机会，而是直接传入队列的下一个。**<br>

test3:<br>

代码上和2一样，但是输入换成了"1 a b c",得到的输出是1,0。没有预期得到type error,这个0不是ASCII码，换成“你好”也是0。<br>

推论1:**cin>>会自动转换类型，如果无法转换，会返回0。**<br>

如果我输入1 a 2 3，它并不会越位读取2，很好理解，如果越位了那整个输入流就错位了。<br>

推论2:**cin>>不会越位读取，即使输入不匹配。**<br>

缺陷：<br>
* 因为用空格分隔一次性可以输入多个进缓冲区，可能造成缓冲区污染。<br>
* 无法转换的类型没有抛出异常直接返回0，这个可能是一个很大的地雷。<br>

结构解析:<br>

![img_7.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410141315047.png)

你可以在

### 1.3：注释简介/注释关键字嵌套的影响：

![image-20241014131833804](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410141321764.png)

你可以看[1.7.cpp](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/1.7.cpp)中代码块的颜色<br>

从第一个*/开始，到第二个*/之间的内容被作为源码了，我写了一个main，编译并没有出错，只是卡在最后一个无头*/而无法生成输出。<br>

错误信息：`error: expected unqualified-id before ‘/’ token`<br>

![img_5.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410140949547.png)

直接说一下推论：<br>
```cmd
* ""内的/*或者*/会被作为字符串，而非注释关键字。<br>
* 如果""外存在第一个/*.那么""内的*/会被作为注释关键字。<br>
* /*总是会找之后的第一个*/作为结束符,哪怕第一个在""内，第二个在""外，它依然会截断字符串。<br>
* 第一个/*会忽略之后的所有*/，直到找到第一个*/。<br>
```

### 1.4：控制流：

这里的主要说的是`for`。<br>

![img_3.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410151622010.png)

你可以先想一下，循环体内i的取值区间是多少，一共加了多少次。<br>

python以前刚刚学的时候，有很多**左闭右开**的情况，转过来的时候，我有点搞不清楚（a；b；c）内和循环体内的执行顺序。<br>

以及 `i--` 和`--i`这种表达式的区别。我不是很能确定终止值和起始值。<br>

1.12的解法：<br>

i的区间是【-100,100】。一共加了201次。和是0。<br>


一句推论，**起始值**只和初始化(a)有关，**终止值**之和判断语句(b)有关。c内不管是--在前还是在后都不影响for循环的执行。<br>

能不能执行也之和b有关，如果初始值都不满足条件b，那么循环会直接跳过。<br>

![img_2.png](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410151622146.png)

理清之后，这真的很简单，但之前是真的有着大大的困惑。<br>



**i++和++i的区别：**<br>

```CPP
int i = 0;
int j = 0;
cout<<i++<<endl;
cout<<++j<<endl;
```
output:<br>
```cmd
0
1
```

影响到的是中间值，只有在比较高优先级的运算符里才会有区别。<br>

对于for循环，这个区别是没有的，那个只看最终值。因为两者最终值都是递增1。<br>

### 1.4.3:输入不定量的数据/while(std::cin>>value)/cin缓冲区的结束符：

```CPP
while(std::cin>>value)
```
这个语句乍一看没看懂。<br>

因为python里，while后面语句可以是执行语句，但肯定是返回一个Bool值的。<br>

但cin似乎没有明确返回值。<br>

经过验证【可以跳转末尾看几个输入输出示例】，它有两种情况会返回false,第一个是类型转换失败，第二个是队列末尾。<br>

结合我的1.2缓冲区结构来理解。<br>

这个语句是相当于把整个队列都一个一个读进来，读一个，进行一次循环体内容。<br>

另外，这里补充了缓冲区的一个结构。<br>

`缓冲区的结尾`，有一个EOF，或者是回车(\n),和字符串末尾的'\0'很像,表示结尾符。碰到它时，cin也会bool False，退出循环。<br>

另外，这个EOF，windows里是Ctrl+Z，linux里是Ctrl+D。<br>

这是几个输出示例:<br>

```cmd
(base) xnne@DESKTOP-3I1GRP0:/mnt/d/program/CPP-Primer-Note/1-开始$ ./a.out
1 2 0 3 4
0+1=1
1+2=3
3+0=3
3+3=6
6+4=10
//还可以再输入
```

```cmd
(base) xnne@DESKTOP-3I1GRP0:/mnt/d/program/CPP-Primer-Note/1-开始$ ./a.out
1 2 a b 3 5
0+1=1
1+2=3

```
上面都是用回车来触发的，下面是用Ctrl+D来触发的。原本想要ctrl+d后面再输入一些看会不会被忽略，但一按下去就直接打印出输出了。<br>

```cmd
(base) xnne@DESKTOP-3I1GRP0:/mnt/d/program/CPP-Primer-Note/1-开始$ ./a.out
1 2 3 4 0+1=1
1+2=3
3+3=6
6+4=10

```

你也可以去自己运行尝试:[1.16.cpp](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/1.16.cpp)。<br>

`回来补档:`<br>

如果使用了`while(std::cin>>value)`的输入方式，回车无法终止输入流，只能用Ctrl+D或者Ctrl+Z。<br>

所以这个示例[1.18.cpp](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/1.18.cpp)中，hello world都无法被打印出来，需要EOF才能打印。否则看上去就像漏了一行，像个bug。<br>


### if(std::cin>>value)：

有了我们上一小节的cin返回值的铺垫，这个if语句就很好理解了。<br>

它可以用来输入的同时检测输入是否合法，是否转换失败。之前1.2的测试中——in:[1 2 a 4 ],out:[1,2,0,4]，这个0就是转换失败的自动返回值。<br>

但请不要这么写：<br>

```CPP
std::cin>>value;
if(value)
```
因为用户可能原本就要输入0,这样会漏掉0的情况，不能作为输入合法性的判断。<br>


### 1.5:类

作者在[Download](https://www.informit.com/store/c-plus-plus-primer-9780321714114)里提供了源代码的下载地址。<br>

[Sales_item.h](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/Sales_item.h)也包含在其中。我把它放在了github上，包括转换[python的伪代码](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/Sales_item.py)。<br>

因为CPP看起来相当吃力，运算符重载，内联外联，看得一愣一愣的，所以让gpt转了一个python。<br>

我也不禁感到，老师讲的那点CPP估计连皮毛都没到。<br>

python里，有用到一点装饰器的内容，稍微补充一下。<br>

```python
假设类名为Sales_item
@staticmethod #静态方法,寄存于类，但完全不依赖类，可以直接调用，Sales_item.func。access:完全不能类属性参数。
@property #属性方法，可以直接调用，不用加括号，Sales_item.func，access:可以调用类实例化参数。
@classmethod #也是可以直接调用，Sales_item.func，access:只能调用类Init之前定义参数。
```

#### Shell小番外：


**从文件输入重定向**:

为了防止每次运行都要重新输入一大串书号，作者推荐了重定向输入。<br>

```shell
additem < infile > outfile
```
起初< >让我困惑，但后来才意识到这个就只是一个重定向。<br>

对于wsl环境，应该像这样:<br>

```shell
./a.out < infile 
```
对于output,直接打印出来就好。<br>

**自定义bin bash命令**:<br>

另外，还可以更偷懒一些，我希望可以用`cprun cpp_script [< infile]来取代以下过程:<br>

```shell
g++ $cpp_script
./a.out < $infile
```
你可以下载[cprun](https://github.com/MrXnneHang/CPP-Primer-Note/blob/main/1-%E5%BC%80%E5%A7%8B/cprun),然后`chmod +x cprun`,赋予执行权限，然后`sudo mv cprun /usr/local/bin/
`，这样就可以直接在任何地方使用cprun了。这个bin的位置可能会因为发行版而不同，我是ubuntu。<br>

#### 1.20题:

![image-20241016090545911](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410160905155.png)

如果是以前的我，会在类里面定义一个赋值函数，一个打印函数，然后在输入的时候，会先cin到临时变量，然后再赋值给类的成员变量。<br>

现在一看，我之前简直就是个傻子。<br>

不重载运算符，每个类的实例，以及类的实例与实例之间就像有天堑，不能比较，不能赋值。所以我以前不喜欢用CPP,而觉得python方便。<br>

现在再看只是我没学到点上，它也可以这么优雅和简短——<br>

```C++
Sales_item item
while(std::cin>>item)
{
    std::cout<<item<<std::endl;
}
```

哪怕仅仅只是基础部分，这个作者依然让我拓展了非常多的认知。<br>