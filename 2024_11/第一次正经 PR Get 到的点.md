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

尝试新建 feature 分支，然后修改，以 feature 分支发起 PR，Merge 后，我们可以直接删了 feature 分支然后再同步主分支，merge 这种费力的事情让仓库维护者做就好了。<br>

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

`gpt say yes`:<br>

> 是的，你的理解是正确的。在`__init__`.py 中使用类似[('__invert__', 'bitwise_not')]的映射，通常是为了实现 API 复用或别名映射。这样做的目的是通过将一个方法（比如 bitwise_not）映射到另一个方法（如`__invert__`），在类的外部或调用接口中实现不同的访问方式。<br>
> 这种写法允许你在不改变底层实现的情况下，暴露不同的接口。例如，你在 Tensor 类中定义了 bitwise_invert，并且将其通过映射与`__invert__`相关联，这样用户可以通过调用 bitwise_invert 来触发 bitwise_not 的实现，而不需要直接暴露原始方法。这实际上是通过映射表来“复用”API，从而使 API 更具灵活性和可访问性。<br>
> 此外，仿照`__invert__`的写法并在 Tensor 的父类中定义 bitwise_invert，是为了确保你在类层次结构中复用并正确挂载该方法。这样可以保持一致的接口，确保方法的行为和返回值一致。<br>
> 总结来说，[('__invert__', 'bitwise_not')]这样的写法本质上是在实现 API 的别名映射，是一种常见的 API 复用手段，有助于通过多种方式提供相同的功能。<br>

这个需要进一步确认和验证，以及，也多看看别人的复用方法，比如[【Hackathon No.10】新增 LogNormal API #46426](https://github.com/PaddlePaddle/Paddle/pull/46426)、[【Python array API standard】 Support Tensor.mT in eager/static mode #68833](https://github.com/PaddlePaddle/Paddle/pull/68833)<br>

但今天后面用的是另外一种复用方法:<br>

**找出 bitwise_not 函数，定义同级 bitwise_invert,在函数内完全复用 bitwise_not,最后在`__init__.py`中进行 import 和['bitwise_invert']。**<br>

这么做的一个好处是，显示定义`bitwise_invert`，而不是让人绕一圈。但代码量多了不少。<br>

似乎这俩各有用处，前一种我刚刚看到时一愣一愣的，后一种好理解很多，但代码更多。<br>

为了方便地称呼这俩，我就都命名一下，前一个就叫`别名映射`，后一个就叫`显式定义`。<br>

### 尝试用 feature 分支发起 PR

paddle 的 CI 真的太慢了，四五个小时都跑不完，而在 develop 分支里面发起 PR 后,我是一动都不敢动。<br>

因为 push develop 分支后会直接跑进 PR 里，同时让 CI 重新运行。<br>

但如果说这个时候`git chekout`来开新分支，又太晚了，因为这时候开分支会把我 PR 的内容带进新的分支里，影响另外一个 PR。<br>

理想状态是，一个 develop 分支负责同步远程仓库，想写内容了就开 feature1 分支,写完了 PR 后删掉 feature1 分支，然后再同步 develop 分支。期间如果 CI 卡住了，我们还可以再开 feature2 分支写第二个 PR。<br>

而且切换分支还可以`git reset --hard`本地快速恢复另一个分支的状态。<br>

or,我应该现在自己的仓库里试试，我说虽然说的天花乱坠，但实际上我还没试过。(>\_<)<br>
