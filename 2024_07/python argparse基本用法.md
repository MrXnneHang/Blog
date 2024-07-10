## python argparse基本用法



### 1.初始化，读取参数。

```python
import argparse

parser = argparse.ArgumentParser(description="读取 MySQL 数据库中的数据")
args = parser.parse_args()
```



### 2.参数的几种指定和传入方式。

* default

```python
parser.add_argument("-d","--database", default="users", help="指定要连接的数据库名称")
```

```cmd
python insert.py --database users
python insert.py -d users
```

* store_true/store_false

```python
parser.add_argument("--use_pen", action="store_true", help="如果指定了，则会使用pen")
```

```cmd
python insert.py --use_pen
```

store_true如果被指定了，则会传入True value给args.use_pen.

![Snipaste_2024-07-10_16-56-10](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407101703172.jpeg)

store_false则相反。

* choice

```python
parser.add_argument("direction", choices=["left", "right", "up", "down"], help="移动方向")
```

```cmd
python script.py left
```

![Snipaste_2024-07-10_17-00-17](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407101704720.jpeg)

不过可惜的是choices似乎不支持整型的输入。

![Snipaste_2024-07-10_17-01-42](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407101704439.jpeg)

但对于这个可以使用指定输入类型来解决。

* type=

```python
parser.add_argument("direction", type=int, help="移动方向")
```

```
python tmp.py 1
python tmp.py "1"
```

![Snipaste_2024-07-10_17-02-19](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407101710778.jpeg)

你也许注意到我在type=和choice中都不再使用--direction的写法。这是因为当我没有指定--的时候，它就是一个位置参数。我可以按顺序传入。当我有多个位置参数的时候我就一次填入参数值而不用填参数名。

### 总结：

> 较推荐的是使用--。
>
> 比较推荐传入都使用字符串，而不是没带引号地传入。
>
> 对于True False的建议使用store_true(传入就是true，不传入就是false)
