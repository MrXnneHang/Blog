```python
for i in range(len(self.chat_windows)):
    self.chat_windows[i].send_button.clicked.connect(lambda idx=i: self.send_message(idx))
    self.chat_windows[i].message_line.returnPressed.connect(lambda idx=i: self.send_message(idx))
```

### Issue:

回车可以正常发送消息，但是send_button不生效,而且会打印奇怪的值。

**原因:** 这个问题通常是由于 lambda 表达式中的闭包问题引起的。你可以通过使用 `functools.partial` 来避免这个问题。以下是修改后的代码：

```python
from functools import partial

for i in range(len(self.chat_windows)):
    self.chat_windows[i].send_button.clicked.connect(partial(self.send_message, i))
    self.chat_windows[i].message_line.returnPressed.connect(partial(self.send_message, i))
```

### 关于闭包

闭包是一个函数对象，它能够记住并访问其创建时的环境变量，即使在这个函数在其词法作用域之外被调用时也是如此。闭包通常用于函数内部定义另一个函数，并且这个内部函数可以访问外部函数的局部变量。

也就是说，如果不清楚适应范围，随意地使用是不安全的。应该被替换。