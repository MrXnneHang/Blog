# 第一次正经 PR Get 到的点 | API 复用的两种方法:别名映射和显式定义

今天这个比较零散。<br>

当真正尝试融入大型开源仓库还是会出现一些问题,即使已经事先遵循规范。<br>

## 大忌：编译的仓库和 commit 的仓库不是同一个仓库。

> `IMPORTANT`:<br>
>
> 请保证你修改的仓库和你编译的仓库是同一个!!<br>

请不要在本地存两个仓库，我最开始学 docker 编译 paddle 的时候，我直接 clone 了 Paddle 远程仓库。然后后来，我开始修改源代码,我又 Fork 并且 Clone 了个人仓库 Paddle-Local。但因为我觉得编译太浪费时间我就直接用 Paddle 仓库做编译，然后编译测试成功后在 Paddle-Local 那边修改代码然后 push。<br>

隐患从这里埋下。<br>

我 clone 两个仓库的时间是不一样的，大概差了两天。期间我要 PR 的文件也被修改过。<br>

然后在一次我尝试先修改了较新的 Paddle-Local 的文件后，然后犯懒粘贴到 Paddle 里，我就发现编译不成功了。<br>

在经过很久很久的排查，我发现了端倪，两个文件初始状态压根不一样长。较新的 Paddle-Local 比 Paddle 那个文件长 3 行。而且又是`__init__.py`文件，于是可以肯定的是，我少缺少定义了，至少少了三个函数。【具体哪三个我也不清楚，因为那个文件太长了】。<br>

如果不是那鬼使神差的一个强行复制粘贴导致的编译失败，我可能现在还在用那种一个仓库编译，一个仓库修改的方式。实际上这逻辑有问题，那边编译通过，不一定这边编译就能通过，何况这两个仓库还不是完全一样的。<br>

现在正在漫长的编译 Paddle-Local。不过还好发现得早。以及之前编译完后写的[Deepin 使用 docker 编译 paddlepaddle](https://xnnehang.top/blog/159)让我耗时大大减少，至少只要复制粘贴一通然后等两次就好。记录博客是各好习惯。<br>

## Tips

### CI 中断的很奇怪。

`Details` 页面里右上角可以手动重新运行 CI,有时候重新运行一下就 pass 了。<br>

### 避免使用主分支进行 PR:

主分支在同步 fork 的时候会生成一些 commit message,比如`Merge branch 'develop' of。<br>

我们应该尝试同步主分支，但经常的同步主分支会让 PR 里面出现很多那种无效 Message,实际上很影响观感。<br>

### 从被堵塞的主分支中跳出来

上面提到了避免使用主分支进行 PR， 因为影响观感，还有一点就是 PR 过程中 CI 和 Review 的流程很长。期间提交到主分支的 commit 都会自动进 PR,导致不敢改其他内容，就会形成一种“堵塞”的现象。<br>

这个时候再新建 branch 也会碰到包含已经提交到上一个 PR 的 commit 的问题。<br>

尝试在 github 中 fork 新分支，但是会显示`No available destinations to fork this repository.`

这个时候，我的解决方法是:<br>

`remote add upstream(PaddlePaddle/Paddle)`->`fetch upstream develop`->`checkout upstream/develop`->`checkout -b feature_branch`。<br>

值得注意的是，`git checkout upstream/develop`后并不会直接 switch 到这个分支，而是会进入一种分离头指针的状态。<br>

具体分离头指针是什么，我放在下面的`深入了解`里。<br>

### 写一个漂亮的 PR Description

> Tip:<br>
>
> 描述里可以链接上相关 issue、PR，比如本 PR 要链接任务 issue #69082、中文文档 PaddlePaddle/docs#6932<br>
>
> 中文文档那边相应链接过来<br>
>
> 这样相互之间引用关系会很清晰～<br>

我尝试 mention，但是尝试了半天也没搞懂像这种渲染的效果是怎么做出来的。

![image-20241108133415924](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411081334795.png)

然后好像有人帮我改了一下，我得到了它的源码：<br>

```markdown
- task: #69082
- docs: PaddlePaddle/docs#6932
```

非常感谢=-=。<br>

## 还需要深入了解的：

### `__init.py` 中的[（'`__invert__`','`bitwise_not`'）]是否是一种 API 复用的写法？

我`Find in files`最终也没找出来更多的`__invert__`的相关定义，唯一挂钩的点似乎就是上面那个。<br>
而我在尝试新建 API`bitwise_invert`的时候，也尝试完全仿照`__invert__`的写法,只在 class Tensor 的抽象父类里面紧跟`__invert__`定义了`bitwise_invert`,然后同样挂到了:<br>

```python
[（'`__invert__`','`bitwise_not`'）,
('`bitwise_invert','bitwise_not')]
```

然后编译出来后就能够访问到`bitwise_invert`类了。而且使用方法，返回值和`__invert__`一模一样。<br>
似乎，[('A','B')]在`__init__.py`中就是一种 API 复用的写法？<br>

`gpt say yes`<br>

这个需要进一步确认和验证，以及，也多看看别人的复用方法，比如[【Hackathon No.10】新增 LogNormal API #46426](https://github.com/PaddlePaddle/Paddle/pull/46426)、[【Python array API standard】 Support Tensor.mT in eager/static mode #68833](https://github.com/PaddlePaddle/Paddle/pull/68833)<br>

但今天后面用的是另外一种复用方法:<br>

**找出 bitwise_not 函数，定义同级 bitwise_invert,在函数内完全复用 bitwise_not,最后在`__init__.py`中进行 import 和['bitwise_invert']。**<br>

这么做的一个好处是，显示定义`bitwise_invert`，而不是让人绕一圈。但代码量多了不少。<br>

似乎这俩各有用处，前一种我刚刚看到时一愣一愣的，后一种好理解很多，但代码更多。<br>

为了方便地称呼这俩，我就都命名一下，前一个就叫`别名映射`，后一个就叫`显式定义`。<br>

### 动态图测试和静态图测试：

paddle.diable_static() 和 paddle.enable_static() .<br>

"运行即定义"（静态图）,"定义即运行"（动态图）<br>

静态图有点像预编译，动态图有点像按步解释执行。<br>

静态图会大概创建一个计算路径，然后一次性计算，而动态图则是灵活地一边计算一边补充计算路径。<br>

大概会有什么区别，静态图定义计算路径的时候，应该会创建很多 tensor，产生很多变量复制的现象，即使是同一个 tensor，也会有很多个副本，这点开销比较大。<br>

而动态图灵活的计算可以自由选用是否要进行原地操作，直接修改原变量而不需要创建新变量，这样开销会小很多。<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411081611572.png)

![image-20241108161256211](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411081613447.png)

这里我们用 CNN 和 RNN 的计算来说明这两种图的区别。<br>

对于 RNN,假设 xn 的长度代表句子的单词数，比如 xn=3，这个输入的句子可以是，比如"I love you."。<br>

而对于 RNN 这种每次输入长度不固定的情况，静态图很难一次性定义计算路径，或者说做不到。这时候动态图就很有优势。<br>

而对于 CNN,每次输入的尺寸，输出的尺寸，中间经历的卷积路径固定，它就适合静态图，甚至静态图可以比动态图快的多得多。<br>

另外，静态图还有诸多限制，因为计算的延迟性，Tensor.numpy() 不能在计算图中调用，之前因为这个测试失败了一次。<br>

但我们这里测试 API 时，不需要考虑太多，只需要验证几点，API 操作的动态图计算结果是否等于静态图计算结果。动态图和静态图的计算结果是否符合预期。<br>

这里放两段代码来说明这两种图。<br>

```python
def test_bitwise_not(self):
    paddle.disable_static()
    x_np = np.random.randint(-100, 100, [2, 3, 5]).astype("int32")
    res_np_b = ~x_np # numpy位取反结果
    res_np_c = paddle.bitwise_not(paddle.to_tensor(x_np)) # paddle 动态 bitwise_not结果
    res_np_d = x_np.__invert__() # numpy __invert__结果
    res_np_e = res_np_d # numpy __invert__结果
    paddle.enable_static()
    with paddle.pir_utils.IrGuard():
        main_program, exe, program_guard = new_program()
        with program_guard:
            x = paddle.static.data(name='x', shape=[2, 3, 5], dtype='int32')
            b = ~x # paddle static 位取反结果
            c = x.bitwise_not() # static bitwise_not结果
            d = x.__invert__() # static paddle.Tensor.__invert__结果
            e = x.bitwise_invert() # static paddle.Tensor.bitwise_invert结果
            (b_np, c_np, d_np, e_np) = exe.run(
                main_program,
                feed={"x": x_np},
                fetch_list=[b, c, d, e],
            )
            np.testing.assert_array_equal(res_np_b, b_np) # 静态位取反是否与numpy位取反结果一致
            np.testing.assert_array_equal(res_np_c, c_np) # 动态paddle.bitwise_not是否与静态Tensor.bitwise_not结果一致
            np.testing.assert_array_equal(res_np_d, d_np) # 静态paddle.Tensor.__invert__是否与numpy __invert__结果一致
            np.testing.assert_array_equal(res_np_e, e_np) # 静态paddle.Tensor.bitwise_invert是否与numpy __invert__结果一致
```

我这一次添加的是`bitwise_invert`,以及`bitwise_invert`和`Tensor.__invert__`都是复用`bitwise_not`.<br>

审核人说这里只能测到静态图的 bitwise_invert，动态测不到.<br>

后面又加上了:<br>

```python
class TestBitwiseInvertApi(unittest.TestCase):
    def setUp(self):
        paddle.disable_static()

        self.dtype = np.int32
        self.shape = [2, 3, 4, 5]
        self.low = -100
        self.high = 100
        x = np.random.randint(self.low, self.high, self.shape, dtype=self.dtype)
        self.x = paddle.to_tensor(x)
        self.expected_out = np.bitwise_not(x)

    def test_bitwise_invert_out_of_place(self):
        result = paddle.bitwise_invert(self.x)
        np.testing.assert_array_equal(result.numpy(), self.expected_out)

    def test_bitwise_invert_in_place(self):
        x_copy = self.x.clone()
        x_copy.bitwise_invert_()
        np.testing.assert_array_equal(x_copy.numpy(), self.expected_out)
```

从后面加上了的代码可以判断，`测动态图`本身是测试**在动态图计算下，paddle.bitwise_invert 结果是否等于 numpy.bitwise_not 结果。**<br>

在原本第一个里面，确实只有静态图下和 np.bitwise_not 的比较。<br>

以及最初让我困惑的点是，在最初的代码里，为什么在动态图里用 paddle.bitwise_not(),而在静态图里却用 Tensor.bitwise_not()。<br>

后面和 gpt 再进行了一阵子讨论，可以确定的是，不管写成 paddle.bitwise_not() 还是 Tensor.bitwise_not(),都是合法的，它们在动态图和静态图下也不会有什么区别。并且都测试的是同一个 kernel，**而且也只需要测一遍，不需要把每个别名都来一遍。**<br>

### Git Detached HEAD:

在试图切换到远程分支时，会出现`detached HEAD`的情况，它是个中间状态，就像只读的文件一样。你可以直接`git checkout -b new_branch_name`来复制一个副本，然后副本是可以修改的。（实际上它的存在方便了分支创建）但你对远程分支的所有修改均不会自动保存，commit 在切换回本地分支后会丢失。<br>

它的特性:<br>

- 临时性，如果你在这里 commit 后没有进行 merge，那么这个 commit 会被丢弃，不会被保存。<br>
- commit 无法直接 push 到远程仓库，因为实际上你的 commit 分支和你 HEAD 指向的分支是独立的。<br>

以前在不小心进入这个状态后会非常害怕，不知道怎么退出来，好像整个 git 仓库都废了，然后甚至把.git 都删了然后重新 clone。(不愧是我)<br>

大概知道这个特性后，只需要切换回本地分支或者创建一个 feature 就能恢复正常了。<br>
