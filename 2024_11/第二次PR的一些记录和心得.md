# 第二次 PR 的一些记录&之后的一些安排

## docstring:

paddle 的 docstring 是遵守 rst 格式的，并且在 CI 中会进行 docstring 的提取和生成 en_doc 的 preview，如果 docstring 不符合 rst 格式，会导致 CI 失败，无法合并 PR。<br>

有多严谨呢，它会对 example code 进行执行，如果少 import 了一个包，也会导致 CI 失败。<br>

[https://github.com/PaddlePaddle/Paddle/pull/69256](https://github.com/PaddlePaddle/Paddle/pull/69256)<br>

这里我多次和 docstring 博弈，如果有机会，可以问问能不能在本地跑 docstring 的 CI，这样可以提前发现问题。<br>

以及这里的建议是，复制别人的 docstring，然后修改内容，这样可以避免一些格式问题。<br>

## CPP kernel 和算子

我注意到，比如 Tensor.biwise_not()这样的函数，它本身只是在 python 中调用了一个 C++的函数，似乎它被称作 kernel，需要了解一下怎么定义和映射这个 kernel,因为似乎 CPP 里面的函数并不叫这个名字，和 torch 的实现略有不同。<br>

我找到了新建 bitwise_op 的 PR:[Add New OP and API: bitwise_and/or/xor/not #33524](https://github.com/PaddlePaddle/Paddle/pull/33524/files)<br>

之后我应该会开一个新的博客来研究一下它添加的代码，做一些 API 复用的任务能够让我快速了解 Paddle 的工作流和规范，但是，我希望适度地深入一些，尽管我的 CPP 的 coding 能力一塌糊涂。<br>

## 静态图和动态图的测试：

不管啥时候都保证两个都测一遍吧。<br>

---

## 最后：

整体而言，这次第二个 PR 和第三个 PR 都是重复性的，我不反对帮别人把这些杂活都做完，但是得留一点给之后训练营的人做。<br>

所以，暂时先看看这个 bitwise_op 的 PR 吧。<br>
