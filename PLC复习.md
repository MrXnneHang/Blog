# PLC 复习

# 电路控制(主电路/控制电路)

## 概念：

![image-20250108101222247](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081015807.png)

1.交流1200，直流1500.我记得交流直流有个根号三的关系。而且直流应该比较大，但能量似乎差不多。

2.控制系统和执行系统。如果考试也这么送分就好了。

3.原来触点也叫执行机构，maybe 常开和常闭？ask

![image-20250108101931163](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081019461.png)

和电器串联过载保护。

熔断器的额定电流可以选择为被保护电路额定电流的1.25～1.5倍。

并且还得考虑能够短时间内承载大的启动电流。

![image-20250108102248749](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081022127.png)

电磁继电器以小控大，接触器直接控制。接触器的额定电流通常略大于工作电器，而电磁继电器可以以非常小的电压工作。

![image-20250108102505793](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081025535.png)

不会立刻响应，会根据设置的延时等待一段时间后触发。

分为通电延时，断电延时，瞬时动作。

![image-20250108102728119](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081027536.png)

不能，因为热继电器的动作需要对热量的持续积累，积累时间比较长，它不会对瞬时大电流做出立即响应，而等待的时间足够烧毁电器引起火灾。

热继电器更适用于长期高负载的保护，而不是短路保护

连接方式，将常闭触点串联到电路中。

![image-20250108103143231](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081031476.png)

指令电器。

发出控制命令的电器或者设备。

开关，按钮，PLC模块。分为人为操作的，被动操作的和自动操作的。

## 主回路和控制回路

![image-20250108115921747](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081159504.png)

FR,热继电器。

SB2闭合，M1转，KM1常开闭合，KM1持续转。

此基础上闭合SB4,然后闭合，KM2转，KM2闭合，KM2持续转。

只有1启动，2才能启动。SB1,SB3用于停止。

![image-20250108115729550](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081157689.png)

必须关机的正反转。

SB1总开关，按下切断整个电路。FR在长期过热，切断整个电路。

SB2常开触发闭合，KM1 ON ,  PWORK ALONG，这时候KM2支路KM1 被切断，不能直接反转。

操作是，按下SB1,重置电路，再按SB3。

线圈前的KM1,KM2常闭开关是对电动机的保护，避免直接反转。

## ps这个图似乎也叫“起保停”。“正停反”

![image-20250108121149815](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081211014.png)

主电路，辅助电路。

主电路，控制电路。

![image-20250108121354184](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081213328.png)

KM是接触器而不是继电器。

接触器在于直接响应，响应快。

但是，缺点在于，不能延时控制。需要配合延时模块。

一直误以为这个有线圈就是继电器。

另外，这个接触器的开关叫做接触器常开，常闭。都是被动触发的。线圈叫做接触器线圈。

不过直接叫常开常闭也可以。反正接触器线圈通电了就直接触发，常开闭合，常闭断开。

## 自锁 ，这个图似乎也叫“正反停”

![image-20250108123817514](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081238366.png)



SB1总开关（按钮）。

SB2，SB3按钮。

这里有必要说明一下一直依赖对于按钮的误会。按钮是那种，按一下触发一下松开就断开的。而开关是闭合了就不会主动断开的。

理论来说，即使没有KM1，把SB2找块石头压住不放也会达到一样的效果（>ω<）

所以，这里SB2,SB3自锁或者互锁（这不重要）的含义就是：

按下后，切断对方支路，使对方线圈一瞬间失电，然后断开KM常开，然后整条路就失活了。

这里依然有一个缺点，就是费机械。

主电路的正反转在一瞬间切换，而没有先等待速度降下来后再切换。会对转轴产生很大的压力。



## 顺序启动，顺序停止

![image-20250108130157091](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081301514.png)

我发现我前面只有一个“正停反”，以及一个并联控制无关的电动机。

而这里还有一个电动机只有在另外一个启动之后才能启动的i情况。

这个一般都是由控制电路决定的，我们主要看控制电路。

FR1,2，任何一个长期高负载都会切断整个系统。

SB1切断KM1

SB2启动KM1

KM1常开可以自行维持KM1工作，KM1线圈的电动机可以单独工作。

那么KM2支路的电动机应该就是附庸了。

可以看到在启动上，只有KM1保持运转，KM2支路的常开KM1才会闭合。

SB3可以单独断开KM2，SB4用于启动KM2并且保持。

关键在于KM2保持后，SB1被短接了，不能直接停下KM1，除非过载切断。

SB2也因为KM1的保持被短接，所以KM2不停下来，KM1就不能停。

结合前面的KM1常开，这个图就做到了。

只有KM1启动，KM2才能启动，只有KM2停止，KM1才能停止。

有点麻烦啊。用电路做的话。期望后面的PLC能稍微在逻辑上好理解一些。

关键在于 **用KM1常开短接SB2**, **用KM2常开短接SB1**， **用KM1常开控制KM2支路**.

个人觉得还是由必要做一个SB0放在F0后面用于紧急制动，一次性停下两个。

## 电动机保护

![image-20250108131306830](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081313507.png)



我一直以为电动机保护是指的是机械保护，减少电动机机械损耗。但看起来并不是这样子的。

另外，我还一直以为“起保停”的保指的是保护，原来是保持。

误会还是挺多的，解开的时候还是挺爽的。

### 过载和短路的区别。

过载一般是长期性，超一些，就像超频超一些。

而短路通常是，你的内存条没插进去。(=ω=)

一个是在长期损害你的电脑，一个只要两三秒，就能闻到金手指的味道了。

其他感觉没啥重要的，不看了。

## 希望送分题不要丢了

![image-20250108131804876](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081318440.png)

这里又有两个误会。

a只是继电器线圈，而不是继电器。

b,c是继电器的触点，而不是开关，虽然它确实起到开关的作用。

完整的回答是这样的

1.通电延时时间继电器。

2.线圈

3.常开触点

4.常闭触点

a我的记法是电磁通量，进去了就是in就通电。如果全黑是out那是断电。

触点我的记法有点污，就不写了。

![image-20250108132743072](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081327658.png)

延时都是单方面的。

或者说每种时间继电器只能延时一边，而对另外一边的动作则是瞬时的。

![image-20250108133049054](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081330609.png)

如果你需要练习和验证。

## 三相异步电动机降压启动。（我们的星三角即将登场）

什么是异步电动机，前面我们看的都是异步，同步的话转不起来的，异步只是为了让它借助磁场转起来。

### 串阻减压启动

![image-20250108134510706](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081345532.png)

我一开始疑惑，为啥它不直接在KM1前面接一个KM2常闭。

后面发现是KM1一断，KT支路也因为KM1常开断了。

再次强调：

![image-20250108134730581](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081347758.png)

这是我们的按钮，一直按的那种。

![image-20250108134801936](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081348239.png)

这也是。

![image-20250108134817489](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081348772.png)

这个叫触点，只闭合一瞬间的那种。

![image-20250108134847822](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081348016.png)

这样的是我们的开关，一直接通和断开的。

![image-20250108134929406](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081349190.png)

再接一个KM2常开开关是可以解决的。

但是，这也是让我感到不舒服的一点，被大佬传染了强迫症。

怎么说呢，这个辅助电路就像在编程，而这个修修补补的方法是最烂的实现方法。

偏偏，它似乎只能，或者说我只能这么实现。

### 星三角减压启动

![image-20250108135446237](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081354696.png)

一开始以为这个主电路没有负载，后面发现M就是负载。我以为他是电阻。

我们需要一个骰子(似乎也叫色子)

![image-20250108135731414](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081357940.png)

我们看这个六点,三相异步电动机上面就有这么六个借口.

而把原本接进来的线接到出去的那个口上似乎就能实现三角和星星的切换(我个人有点难以理解,因为电动机不应该左右对称的吗?ω?).但是我似乎不考主电路,我们着重看辅助电路.

![image-20250108140118350](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081401117.png)

FR长期过载切断.

SB1按钮切断整个电路,舒服了.

SB2被KM1短接,保持.KT计时,KM3通,KM3常开断开,KM2支路切断,等待T

KT触发,KM3断,KM3常闭恢复,KT常开触点闭合一瞬,KM2通,KM2保持,

启动时,KM1,KM3通,KM2断.

然后变成KM1,KM2通,KM3断.图看懂了,但是还是不是很懂三角在哪里,先跳了.



## 反接制动和能耗制动

![image-20250108141025265](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081410884.png)

![image-20250108141039003](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081410556.png)

一个直接反向干,一个加电阻.

了解一下定义就好了.

## 练习(明天早上试试)

![image-20250108141150072](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081411503.png)

![image-20250108141205803](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081412487.png)

![image-20250108141220753](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081412561.png)

这些我都看过且看得懂,但考试要画,所以还是很有必要实战一下.

# PLC(可编程控制器)部分

## 概念

![image-20250108141437671](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081414063.png)

1.可编程控制器（Programmable Controller）简称PLC

2.轻量,成本低

抗干扰能力强

减少了工作量，维护方便，易于修改调试,

功耗少、成本低、水平高

![image-20250108142620536](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081426524.png)

我简直太赞同了.

**➢输出相对于输入有滞后，不适于快速反应的要求。 **

3.![image-20250108142740691](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081427244.png)

超小型化、超大型化。

通讯化、网络化

编程语言化

4.![image-20250108143135854](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081431171.png)

硬件系统和软件系统,说实话和树莓派挺像的

![image-20250108143207218](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081432398.png)

输入,逻辑,输出

![image-20250108143325936](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081433498.png)

跟树莓派比起来,多了一个编程器.

应该就是用来处理梯形图,相当于一个硬件形式的解释器.

看起来很完备,但是我用的那个一用就觉得它这东西没有内存.虽然这里写着存储器.

但是变成起来那就是完全的if,else,if,else嵌套,看着巨恶心.

后面会接触到的.

5.工作原理通微型计算机类同,它把原来需要接线控制的转换成通过编程控制.

6.微型,小型,大型,超大型.

主要都在IO点和计算能力上,越大成本越高,根据任务适配即可.

7.硬接线和软编程的区别.

在修改性和维护性以及移植性上区别很大.

## 喜闻乐见送分环节2

![image-20250108143056275](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081430990.png)

ACD,A

## 开关量输出

![image-20250108143748540](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081437742.png)

![image-20250108143912618](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081439158.png)

场景更重要.适配交流直流,混合,本身和电磁铁没啥区别,微型的,依靠磁通量变化工作.

![image-20250108144026252](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081440733.png)

晶体管是单向流通的,只直流,很好理解.

![image-20250108144102942](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081441105.png)

整个应该就是CPU的类型,它只能交流.



## 数字量和模拟量:

![image-20250108144735656](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081447226.png)

（A/D，D/A）



## 有的PLC可以在直流下工作.

![image-20250108144825910](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081448155.png)



## 从控制电路到PLC梯形图的过渡

![image-20250108144937898](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081449688.png)

圈圈是输出.

开关只有常开常闭.不分按钮,触点开关.开了就是开了,闭了就是闭了.

![image-20250108145126353](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081451510.png)

左->右,上->下

![image-20250108145158457](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081451930.png)

线圈只能用一次.END.

## 指令表

和汇编类似.一条是一步,一步一步执行,所以,写个for循环你懂的,连goto都没有的.

![image-20250108145434876](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081454374.png)

没有goto是我不能接受的.

## 送分环节3

![image-20250108145742939](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081457418.png)

输入输出的IO点位总是相等的.

16就是I8O8,32,48一次类推.

晶体管和继电器那个能记也稍微记一下.

![image-20250108150101912](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501081501143.png)

不要被ESS骗了,那个是不重要的.

ET,扩展,晶体管.16I,16O.

----



## 后面其实就是编程了,会编程就行了,概念就不重要了.

## 比较重要的是有一个顺序控制和顺序功能图,这俩学会基本就能去考试了

## ======================

## 送分环节4

![image-20250109003246686](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090032507.png)

![image-20250109003315688](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090033534.png)

![image-20250109003756047](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090037321.png)

和周期无关，无需自行脑补。

![image-20250109003828268](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090038139.png)

![image-20250109003950946](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090039180.png)

计时器不会重置只会停止。下次接着计时。

## 定时器触发的信号不会保持

![image-20250109004130476](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090041025.png)

![image-20250109004201190](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090042418.png)

定时器T0和按钮类似，给开关的信号都是只持续一瞬间就断开。得一直通电，才会一直使得T0常闭打开。

![image-20250109004908092](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090049674.png)

我以前有个误解是接触器收到触发信号后会反转，然后等待下一次信号。但实际上这个反转也只维持一瞬间，除非线圈保持通电。而线圈失电后，它会恢复原状。

![image-20250109005433016](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090054046.png)

我们再看这个，实际上我就又发现之前的误解了。

区别在于，T0先触发，然后持续多久，这个是定时器的控制的。

K10是一秒，但感觉跟定义有关。或者和PLC设备有关。

## 顺序启动

![image-20250109010719496](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090107136.png)

Y0,Y2,Y1的常开都是用于保持

X0触发，Y0通，保持，SLEEP 5s,T0触发。

然后是第二行，T0触发一下，无效，第三行T0触发，Y0维持，Y1触发，Y1维持。SLEEP 5s.

T0触发，Y2触发，Y2维持。

T0触发，对Y1无影响。

之后每隔5s，定时触发一次，T0，但是都没有影响。

X1断开,T0断开，Y0断，Y1断，Y2断。几乎同时断开，但依然有顺序关系。

### 思考：

我能否把第二行和第三行即Y1支路与Y2支路调换，应该是可以的。

## 基本指令：

对于前面指令表的补充：

![image-20250109011504561](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090115440.png)

相比下多了F和P,Falling和Prhasing?maybe，反正一个上升一个下降。

## 送分环节5

![image-20250109011749539](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090117973.png)

![image-20250109011829490](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090118972.png)

![image-20250109011930449](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090119974.png)

保个OUT X就好了。LDF,PLF,LDP,PLS类似。

![image-20250109012032587](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090120804.png)

区别在于一个是输入检测，一个是输出。

作为开始端写成LDF,LDP,作为中间端写成ANDF,ANDP.

同理也有ORF,ORP

![image-20250109012201717](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090122327.png)

![image-20250109012225347](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090122908.png)

## LD两次与逻辑块。

![image-20250109012429035](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090124059.png)

分别以X0,X2后面那个干为分隔，分了两个逻辑块，然后进行ANB。

ANB补充：

![image-20250109013207248](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090132350.png)

## SET/RESET

![image-20250109012603271](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090126887.png)

首先是SET和RST

SET=1,RST=0.1得电，0失电。

然后是关于LDP的。

它的定义是，上升沿时，接通。

断开时，会迎来一个下降沿，不会触发。原来是什么状态就是什么状态。而且并不是说不SET，就是RST了。是 **无影响**。

- **当 X0 上升沿到来时，Y0 得电**
- **当 X1 接通时，Y0 失电**



## 送分环节6

![image-20250109013248384](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090132503.png)

俺也不知道这俩什么东西什么关系，但记住这个8应该就行。

![image-20250109013412417](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090134372.png)

MNM,PASS

![image-20250109013525581](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090135359.png)

MPS,MPP是入栈出栈。

11

![image-20250109013644986](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090136266.png)

这和汇编应该就一样了。

中间隔了一个X1,但是后面还要用到单独的X0.

所以先存起来。

后面取出来。

希望不要出入栈多个，然后后入先出（=ω=）。有点涩。

![image-20250109013808727](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090138423.png)

如果没隔一个X1，直接中出即可。

## 按键红绿灯，有时间看一下，优先级在画控制电路之后。

![image-20250109014021302](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090140128.png)



## 功能指令，顺序功能图。

![image-20250109014238054](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090142311.png)

![image-20250109014328292](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090143109.png)

Phrasing还在啊。

## 送分环节7

![image-20250109014433060](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090144879.png)

![image-20250109014653563](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090146152.png)

## CMP (Compare)

![image-20250109014901110](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090149540.png)

![image-20250109014914538](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090149193.png)

小于，等于，大于，取三个。

假设C2=50

M2 = 1 (True)

M2 = 0 False

M3 = 0 False

最终表示，C2 < 100, 写入M2寄存器。

K值决定比较值。

## ZCP

![image-20250109015234714](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090152349.png)

![image-20250109015300726](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090153257.png)

和CMP类似，取连续三个，但是第二个改成区间。

另外，术语置位 means = 1 ,复位 means = 0

## MOV

![image-20250109015426283](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090154159.png)

问题在于，MOV左的是cp，还是mov，在于源操作数会不会被改变。按照定义，不会被改变。

![image-20250109015528660](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090155195.png)

那么这个本质就不是MOV而是CP.

![image-20250109015605111](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090156870.png)

Batch MOV

BMOV D0 D10 K3 mean what?

D是地址，或者说指针，K3意思是三个寄存器。

### 数据寄存器（Data/D) 16/32bit，位寄存器(M)1bit

![image-20250109020024650](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090200256.png)

![image-20250109020044916](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090200383.png)

## Logic操作

XCH，交换数据。

![image-20250109020237416](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090202137.png)

![image-20250109020323619](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090203348.png)

补充一下K1M0,K1表示数据长度为4位置。2^n

K0,K1.

## 送分环节7

![image-20250109020454907](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090204704.png)

8022，咱们混个脸熟。



## =========================

## 终于来到我们的大boss. 顺序控制功能图。

![image-20250109023643168](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090236745.png)

“起保停”

起动不变。

保持，每一个的保持需要上一个的保持。

停止，前面的停止需要后面的停止。

保持和停止相加=全集。



## 顺序功能图(SFC)，规则驾驭脑子的案例。

![image-20250109024135198](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090241836.png)

S,Status

![image-20250109024253235](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090242904.png)

![image-20250109024402804](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090244274.png)

注意if和or的区别即可。

![image-20250109024535476](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090245070.png)

每个方框代表一个逻辑步，一个短横线代表一个判断条件。

如果连着走：

![image-20250109025021466](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090250414.png)

![image-20250109025824523](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090258029.png)

### 选择

![image-20250109025800121](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090258508.png)

按理来说，有规则的话，应该用程序自动转换，而不是让我记住规则然后画出来。

1，遵循下一步是上一步的终止，或者说，必须确保每个分支都进去了，只能通过Mi-1

合并，只要有一个能出来就行。保证有信号出来。或者说也只希望有一个出来。

### 并列（对每个都看作是单独的顺序即可）。

![image-20250109030206503](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090305336.png)

合并的时候，注意，这里每个都作为启动，说明每条路都必须走通才能走下一步。



## 避免线圈输出两次

![image-20250109030714702](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090307992.png)

我记得在PLC图里面，每个Y都应该只出现一次。避免出现两个不同信号同时试图修改Y。

这个没必要改顺序功能图，直接把这俩作为一个条件合并判断即可。

## PS,转换法更好用一些。

![image-20250109032411495](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090324959.png)

![image-20250109032425831](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090324642.png)

![image-20250109032441247](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/25/01/202501090324668.png)



和编程逻辑更像。

如果说起保停是每一步要三层if嵌套，那么这个就是只有一层if。



# 画图练习。

