# 【第二次正经 PR】左操作(`__rshift__`)和右操作(`__rrshift__`) -part1: to_tensor 实现对 int 的类型自动转换

## Refrence:

- [【快乐开源】Paddle Tensor 规范化 #69082](https://github.com/PaddlePaddle/Paddle/issues/69082)

## 发现的潜在问题： 右侧操作数无法正常执行`__rrshift__`和`__rlshift__`操作(一直不调用，调用必抛 TypeError)

### 问题复现:

```python
>>> import paddle
grep: warning: GREP_OPTIONS is deprecated; please use an alias or script
>>> data = paddle.to_tensor([2,4,8])
>>> shift = paddle.to_tensor([1])
>>> data << shift
Tensor(shape=[3], dtype=int64, place=Place(cpu), stop_gradient=True,
       [4 , 8 , 16])
>>> shift << data
Tensor(shape=[3], dtype=int64, place=Place(cpu), stop_gradient=True,
       [4  , 16 , 256])
>>> 1 << data
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python3.9/dist-packages/paddle/tensor/math.py", line 7879, in right_left_shift
    return bitwise_left_shift(x, y, is_arithmetic, out, name)
  File "/usr/local/lib/python3.9/dist-packages/paddle/tensor/math.py", line 7651, in bitwise_left_shift
    return _C_ops.bitwise_left_shift(x, y, is_arithmetic)
ValueError: (InvalidArgument) bitwise_left_shift(): argument 'y' (position 1) must be Tensor, but got int (at /paddle/paddle/fluid/pybind/eager_utils.cc:1330)
```

而 torch 那边:<br>

```python
>>> import torch
>>> data = torch.Tensor([2,4,8]).to(torch.int)
>>> shift = torch.Tensor([1]).to(torch.int)
>>> data << shift
tensor([ 4,  8, 16], dtype=torch.int32)
>>> data << 1
tensor([ 4,  8, 16], dtype=torch.int32)
>>> 1 << data
tensor([  4,  16, 256], dtype=torch.int32)
>>> shift << data
tensor([  4,  16, 256], dtype=torch.int32)
```

看 torch 这个后就逻辑清晰很多了，没有必要强调到底是调用左操作或者右操作。<br>

直接把`<<`和`>>`当作一个运算符号即可，支持广播，不会因为调用是调用左操作或者右操作有什么不同。<br>

虽然结果一致，但区别在于，右操作只在左操作不能正确执行的时候调用:<br>

```python
4 << Tensor([1,2,4]) # 尝试调用4.__lshift__(Tensor),失败，hook消息

# 尝试调用Tensor.__rlshift__(4)【x.__rlshift__(y),Tensor是x,4是y】
# 我们可以在Tensor.__rlshift__中定义对4的类型转换。
# 如果存在一个bitwise_left_shift函数(x << y)->:
# bitwise_left_shift(Tensor([4]),Tensor([1,2,4]))
# 这个bitwise_left_shift在右操作的时候是反着来的(y,x),有点小坑。
# 另外，即使重载成了运算符，比如`<<`，我们也依然可以用x.__lshift__(y)的情况来调用，这么调用会支持输入额外的参数，如果有。比如is_arithmetic有一个符号推断。
```

### 原因：

- `__rrshift__`,`__rlshift__`复用`__rshift__`和`__lshift__`实现
- `__rshift__`和`__lshift__`复用`bitwise_left_shift`和`bitwise_right_shift`实现
- `bitwise_left_shift`和`bitwise_right_shift`实现时，**要求 x,y 都必须是 Tensor 类型**
- 当 x << y , x: int , y: Tensor 时，试图`y.__rlshift__(x)`，会抛出 x 不是 Tensor 类型的 TypeError
- 当试图调用右侧操作时，说明左侧不是 Tenosr,由于底层的`bitwise_left_shift`和`bitwise_right_shift`要求 x,y 都是 Tensor 类型，必定抛出 TypeError。
- `__rlshift__`和`__rrshift__`一直无法被正确调用

### 解决方式:

`__lshift__`和`__rshift__`额外接受 int 并且尝试自动转换为 Tensor 类型，就可以解决这个问题。

参考 torch.Tensor：<br>

```Python
>>> import torch
>>> data = torch.Tensor([2,4,8]).to(torch.int)
>>> shift = torch.Tensor([1]).to(torch.int)
>>> data << shift
tensor([ 4,  8, 16], dtype=torch.int32)
>>> data << 1
tensor([ 4,  8, 16], dtype=torch.int32)
```

它的实现:<br>

```C++
Tensor __lshift__(const Tensor& self, const Tensor& other) {
  Tensor result;
  auto iter = TensorIterator::binary_op(result, self, other);
  lshift_stub(iter.device_type(), iter);
  return iter.output();
}

Tensor __lshift__(const Tensor& self, const Scalar& other) {
  Tensor result;
  auto wrapper = wrapped_scalar_tensor(other);
  auto iter = TensorIterator::binary_op(result, self, wrapper);
  lshift_stub(iter.device_type(), iter);
  return iter.output();
}
```

> Get 一个点，函数重载，相同的函数名，不同的参数类型，调用的时候尝试自动匹配和转换参数类型。死去的 CPP 知识在攻击我。<br>
> 不过这么实现在这里真的好优雅。<br>

## 可选修改方式：

依葫芦画瓢，我只需要模仿出和 torch.Tensor 一样的操作结果即可。<br>

`int-Tensor`、`Tensor-int`、`Tensor-Tensor`。<br>

目前发掘出来的用法就只有这几种。<br>

而`bitwise_left_shift`和`bitwise_right_shift`完美解决了`Tensor-Tensor`的工作，而且其他两个也可以说解决了 90%。<br>

需要考虑是`函数重载`，`Tensor-Tensor`不改变，`额外支持 int 自动类型转换成 Tensor`。因为初始化的是 paddle 的方法，所以应该不能直接调用 paddle.to_tensor。<br>

所以这里就需要几个 PR:<br>

- `to_tensor` 的底层实现。（关注是怎么把 int->Tensor(ndim=0)）,没找到=-=.可以看下面的`full_ad_func`的方法。
- `_C_ops.api`是咋定义和绑定并且使用的.[bitwise_op](https://github.com/PaddlePaddle/Paddle/pull/33524),[bitwise_shift](https://github.com/PaddlePaddle/Paddle/pull/58092)

这么看并不是做不了。<br>

> ps: 这一块 move 到了 [part2](https://xnnehang.top/blog/164) 的部分，因为 PR 里我定义 kernel 的想法被驳回了。<br>

### 修改 bitwise_left_shift 和 bitwise_right_shift，使他增加对 int 类型的支持

不太合理,改动太多了。<br>

### 定义一个 AutoTensor 方法,把输入 int 自动转换为 Tensor

`paddle/fuid/pybind/eager_math_op_patch.cc line1203`:<br>

```C++
// 2. create or get tensor for other_obj
paddle::Tensor other_tensor;
if (has_other_double) {
eager_gil_scoped_release guard;
other_tensor = full_ad_func({1},
                            phi::Scalar(other_double),
                            self_tensor.dtype(),
                            self_tensor.place());
const phi::distributed::ProcessMesh* mesh = nullptr;
if (InputsContainDistTensor(&mesh, self_tensor, other_tensor)) {
    ConvertAllInputsToDistTensor(mesh, self_tensor, other_tensor);
}
```

可以尝试用 full_add_func 转换 int 为 Tensor,可行性更高，而且 torch 也仅仅 support int 型。<br>

这个参见我们的 part2。即使 PR 要求不同，我们先把 PR 交了，然后就接着玩。<br>

## 最终 PR 中的实际做法：

[【Paddle Tensor No.8、9、14、15】为 Tensor 新增`__rshift__`,`__lshift__`,`__rlshift__`,`__rrshift__`](https://github.com/PaddlePaddle/Paddle/pull/69348/files/0ad6ab02cc7278ac45647f5ce018f1368a140d1e..ecbb060723b0c56cf5dfcd3025ab36d30cac3f89)<br>

最后用了 to_tensor .. paddle.to_tensor..<br>

我以为终于可以写算子了，结果又没写成。<br>

## 需要深入了解的：

为什么我会下意识不敢用 to_tensor？主要是因为我在初始化的是 Tensor 的方法，而 to_tensor 是 paddle 的方法，我这里就会担心，我在担心，如果 Tensor 的初始化在 paddle 之后，那么会不会找不到 to_tensor 方法，也确实存在这种风险。我尝试在`python/paddle/tenosr/math.py`中写`from paddle import Tensor`试图进行一个类型判断，然后编译失败了。失败原因是导入 Tensor 时会尝试调用`__init__`，而许多 Tensor 的方法都是在 math 中定义，调用在`__init__`之前，就报错了。<br>

我发现 math.py 里实际上许多函数参数列表里都写着(x:Tensor,y:Tensor)，它是这么导入的:<br>

```python
if TYPE_CHECKING:
    from paddle import Tensor  # 类型检查时导入,注意这里是指静态检查，一般是参数列表的:，而不是if isinstance(x,Tensor)，这个需要直接导入。
```

`TYPING_CHECKING`并不会影响到代码执行逻辑，只是用于静态检查，所有代码执行逻辑都应该直接导入，或者`lazy import`(函数内调用时导入)。<br>

但是，出于性能考虑，任何时候都不应该使用`lazy import`，因为，python 的 import 真的可以很久很久，三四秒那么久，非常影响体验。<br>

按照我的理解划分，python import 大概是因为`编译时`和`运行时`和`类型检查时`,编译时是最严格的，会检查各种相互依赖的`__init__.py`关系先后，容易出现导入未初始化.

随后是`运行时`，运行时相对而言比较没有那么严格，比如，参数列表的(x:Tensor,y:Tensor)并不只是编辑器里面静态检查使用，在运行时，它也会及时反馈，比如我传入一个 int 的类型，那么它可能就会报错，并且提示只能输入 Tensor,这个时候静态检查就自动转换进入了一种`运行时检查`;<br>

再比如，我虽然写着`if TYPE_CHEKING`时导入,我即使偷偷写了`if instance(x,Tensor)`,但是编译时也不会报错，而且也能正常运行而不会报出未定义的错误，有点像延迟导入，但是实际上是一种关键字占位，也是转为`运行时检查`,这个运行是在所有的类都初始化完毕后的。但实际上这有点卡 Python 的 bug,万一哪个版本就把这个隐藏 buf 给删掉了，那到时候就会产生大量的未定义。不建议这么搞。<br>

正经写，`if TYPE_CHEKING`只是作为我们参数列表的一个类型检查，而要偷偷卡上面的 bug。<br>

关于算子，我是不会放弃的，这个只是 part1,我打算自己实现一个 part2 来玩玩，就是按照我最初的想法，写一个 AutoTensor,当然，我觉得我还是先实现一个简单的输出流就好了。<br>
