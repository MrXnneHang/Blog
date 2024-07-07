## 行宽:

don't do these:

1.

```python
# ❌
a = 1;b = 2

# ✔
a = 1
b = 2

# 不要用;把两句合并成一句
```

2.

```python
# ❌
a = b * c + c * d + d * e \
  + c * f + e * g + e * l

# ✔
a = (b * c + c * d + d * e  
   + c * f + e * g + e * l )

# 不要用\来显示续行，使用(),[],内部可以隐式续航。
# 写长的if , while 也是同理。
```

```python
# 长字符串也应该用()续行，会被识别到同一句中。
a = ('你好啊'
     '这是一个很长的字符串.')
print(a)
```

![字符串续行](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910325499.webp)



之后请继续 : [Google python 编写规范](https://github.com/zh-google-styleguide/zh-google-styleguide/blob/master/google-python-styleguide/python_style_rules.rst)