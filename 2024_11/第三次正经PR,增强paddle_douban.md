# 第三次正经 PR: 对 paddle.grad 进行功能增强 - part1 理解自动微分框架

## Refrence:

- [从 0 实现一个极简的自动微分框架](https://blog.nagi.fun/micrograd-tutorial)<br>

## 用代码求导的逻辑:

```python
# Define a scalar function f(x), which accepts a scalar x and returns a scalar y
def f(x):
    return 3*x**2 - 4*x + 5

# Create a set of scalar values to feed into f(x)
xs = np.arange(-5, 5, 0.25)
ys = f(xs)

# We can basically evaluate the derivate here numerically by taking a very small h
h = 0.00000001
x = 3.0
(f(x + h) - f(x)) / h # the function respond positively
```

这个并没有函数的原函数，导数等概念。<br>

这里的导数是一个离散的概念，是针对每一点进行求极限（用一个比较小的数来代表这个极限）。这个极限也被称作梯度(gradient,简称 grad)，很具象，因为本身就像在说这个台阶的倾斜度有多大。<br>

比如我 y 在 x=3 处对 x 求导，那么本身就是上面那个逻辑。我求出`(y(3.000001)-y(3.0)) / 0.000001`。<br>

本身就是一个求导，或者说求偏导。<br>

最简单的计算单元就是这样，它没有更复杂的地方，即使是多变量，每次求导的时候，也是一个"y" 对于 一个 "x" 的形式。<br>

难点在于：

- 对于一个很长的计算链，我们的梯度是怎么计算的，从前往后还是从后往前。又是如何累积。
- 以及每个节点可能并不是只用一次，存在某些节点用一次，某些节点用多次的情况，如何累积梯度。
- 以及，如何自动化地进行上述内容。

## 深层函数求导 - 数值极限求导法 vs 链式求导法：

- 链式求导法则在什么时候起作用，起什么作用。
- 对于一个很长的计算链，我们的梯度是怎么计算的，从前往后还是从后往前。又是如何累积。

### 数值极限求导法：

当我们把上述的计算链适度加长一些：<br>

```python
# Define a scalar function f(x), which accepts a scalar x and returns a scalar y
def Y(x):
    return 3*x**2 - 4*x + 5

def Z(y):
    return y**2 - 4*y

def F(z):
    return z**4 - 5

# Create a set of scalar values to feed into f(x)
xs = np.arange(-5, 5, 0.25)
ys = Y(xs)

# We can basically evaluate the derivate here numerically by taking a very small h
h = 0.00000001
x = 3.0
Y_at_x = (Y(x + h) - Y(x)) / h # Y at x = 3.0
# 14.00000009255109

# 求Z对x=3.0处的导数
Z_at_x = (Z(Y(x+h)) - Z(Y(x))) / h # Z at x = 3.0
# 504.00000759509567

# 对F求x=3.0处的导数
F_at_x = (F(Z(Y(x+h))) - F(Z(Y(x)))) / h # F at 3.0
# 66060290527.34375 似乎爆炸了，不过没事，我们这里讨论的只是计算

print(Y_at_x)
print(Z_at_x)
print(F_at_x)
```

这里可能会有点抽象，我们引用一个简单的 MLP 分类图像的问题来解释一下。<br>

假设这里的 x 是我们的输入图像，z 是我们的输出结果。<br>

我们求导的目的是，更新各个 MLP 全连接映射的中间函数以逼近我们的预期 label(z)。<br>

我们针对不同的组合输入图像（这是一个常量 x）进行求导，得到到 label（这是一个常量 z）的 gradient。<br>

并且 loss_fn 和 optimizer（这个又是另外一个大坑了，咱们先不考虑了,这里会根据 grad，结合 loss_fn）来决定要如何调整 MLP 映射的函数（参数 weight,bias）。<br>

トータル，我们的求导对象，是针对每个输入 x 的，求导结果，是每步计算对应的 grad。一直到我们的输出 z 为止。<br>

以及，可以推理出我们计算过程中并不会进行更新，总是在计算结束后才批量进行更新。<br>

### 链式求导法：

```python
# Define a scalar function f(x), which accepts a scalar x and returns a scalar y
def Y(x):
    return 3*x**2 - 4*x + 5

def Z(y):
    return y**2 - 4*y

def F(z):
    return z**4 - 5

# Create a set of scalar values to feed into f(x)
xs = np.arange(-5, 5, 0.25)
ys = Y(xs)

# We can basically evaluate the derivate here numerically by taking a very small h
h = 0.00000001
x = 3.0
Y_log = (Y(x + h) - Y(x)) / h # Y at x = 3.0
# 14.00000009255109

# 求Z对x=3.0处的导数
Z_log = ((Z(Y(x) + h) - Z(Y(x))) / h) * Y_log # Z at x = 3.0
# 504.00000759509567

# 对F求x=3.0处的导数
F_log = ((F(Z(Y(x)) + h ) - F(Z(Y(x)))) / h) * Z_log # F at 3.0
# 66060290527.34375 似乎爆炸了，不过没事，我们这里讨论的只是计算

print(Y_log)
print(Z_log)
print(F_log)
```

上述数值极限法（我随便取的），从代码上，你似乎看不出来计算量是否有减少，但是如果你从数学上面来看，那么计算量是绝对少了的。<br>

数值极限法，它会产生一个计算的冗余，它总是重复地计算差分。<br>

我们对比一下 `F(Z(Y(x))+h)-F(Z(Y(x)))` 和 `F(Z(Y(x+h))) - F(Z(Y(x)))`.<br>

那个 x+h,它的差分总是被重复计算，有多少层，就重复多少次。假设有三十层，那么这个就传递了三十次，在深度神经网络中，这是一个很大的计算开销。<br>

但是，如果我们使用链式求导，这个计算冗余就直接被消除了。它被优化为一个自函数的(Y_log)。<br>

它的意义在于，把一个深度递归转化成一个简单的乘法。<br>

到这里，我们`对于一个很长的计算链，我们的梯度是怎么计算的，从前往后还是从后往前。又是如何累积和影响的。`这个问题解决了一半。<br>

从我们的推理来看，它应该是从**浅层往深层**的，因为这样才能使用链式求导来减少冗余的。<br>

另外，如果每个计算节点都只使用一次，那么只需要一次遍历，就能够计算出来所有的 grad,准确地说，是我们模型输出`output`对于特定输入 `x`(比如我们 MLP 分类问题中的一张图像)的一个 F_at_x，不需要进行累积。<br>

这样似乎就解决了这个问题。<br>

## 计算节点复用 - 如果一个 y 对应多个 x,那么 y 的梯度应该如何进行累积。（按照我们上面的方法是直接覆盖，不考虑复用的情况的）

我们先思考一下：<br>

- 直接求和
- ~~加权平均~~
- ~~相乘(艺术就是爆炸)~~

pytorch 的`auto_grad`是第一种。大道至简，加权，等于引入人类的思维，等于告诉机器哪部分是重要的，哪部分是不重要的。但是我们实际上希望机器自己懂得这些。<br>

pytorch 的`auto_grad`中有一个从最开始就一直会用到的参数是`requires_grad`，如果一个变量被设置成`reuqires_grad=True`,那么之后，它不仅仅会保存`gradient`的值，同时也会保存计算的链表，记录每一步操作，实际上，直接加和也是会记录在其中的。所以直接求和并不会导致我们的`gradient`被混淆。<br>

并且它还有一个`zero_grad`在每次更新完参数后进行一个梯度的清空。<br>

如果画成图片的话大概是这样:<br>

![alt text](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2915597681.webp)

对于`input layer`，也就是我们的`x`,它自己对自己求导的值就是 1。<br>

对于`hidden layer1`，也就是我们的 y1,y2,y3,y4。它的每个小球的`gradient`都是对三个`x1,x2,x3`对应分别求导并且求和。也就是`y1'x1+y1'x2+y1'x3`，依次类推。<br>

而进入深层比如`hidden layer`中的`z1,z2,z3,z4`。<br>

它需要根据链式求导乘上上一个 layer 的偏导数。<br>

```txt
( z1'y1 * y1'x # x = x1 + x2 +x3
+ z1'y2 * y2'x
+ z1'y3 * y3'x
+ z1'y4 * y4'x)
```

依次类推 z2,z3,z4。<br>

对于最终的`output`,它的 grad:<br>

```txt
f'z1 * z1'y +
f'z2 * z2'y +
f'z3 * z3'y +
f'z4 * z4'y
```

为什么只乘了一个`z1'y`而不需要再乘上`y'x`?<br>

有一瞬间疑惑这个。理由是，它本身已经包含了那些。<br>

你可以到 Part1 的`链式求导法Vs数值极限法`那里去把代码复制粘贴运行一遍。<br>

这样一看，链式求导法则它在计算梯度中简化的计算逻辑简直太多了。<br>

## 如何实现自动化的求梯度？

这就是我们之后的事情了，我们要去读 Pytorch 的`auto_grad`代码实现。

以及我们的实战部分：为 Paddle 的 grad 进行增强。<br>
