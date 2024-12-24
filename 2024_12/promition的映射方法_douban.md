# Type Promotion 的映射方式.

来写点轻松的,我不喜欢总结,更喜欢探索.一直考虑着什么时候把之前的 PR 总结一下,但是感觉就算了.不如写一下这个.<br>

## 什么是 type promotion

Type Promotion 是指在运算时，如果两个操作数的类型不一致，那么会自动将其中一个操作数的类型提升为另一个操作数的类型，以便进行运算。<br>

基本原则:<br>

- 不能产生数据截断
- 不能产生数据溢出
- 对于特别计算不能产生精度损失

如果说需要描述各种数据类型的提升关系，我一开始是想着做映射关系,但想到的是一维的.但那个一个一个列出来,可能会少掉几个也说不定.<br>

看到了这个,第一时间没看懂,但看懂后一瞬间有脑子被凿穿的感觉.<br>

```C++
inline static DataType promoteTypes(DataType x, DataType y) {
  constexpr auto u1 = DataType::UINT8;
  constexpr auto i1 = DataType::INT8;
  constexpr auto i2 = DataType::INT16;
  constexpr auto i4 = DataType::INT32;
  constexpr auto i8 = DataType::INT64;
  constexpr auto f2 = DataType::FLOAT16;
  constexpr auto f4 = DataType::FLOAT32;
  constexpr auto f8 = DataType::FLOAT64;
  constexpr auto c4 = DataType::COMPLEX64;
  constexpr auto c8 = DataType::COMPLEX128;
  constexpr auto b1 = DataType::BOOL;
  constexpr auto bf = DataType::BFLOAT16;

  const int total_type_num = 12;

  // 并且每个 type 本身也都有一个 index.

  static constexpr DataType
      _promoteTypesLookup[total_type_num][total_type_num] = {
          /*        u1  i1  i2  i4  i8  f2  f4  f8  c4  c8  b1  bf*/
          /* u1 */ {u1, i2, i2, i4, i8, f2, f4, f8, c4, c8, u1, bf},
          /* i1 */ {i2, i1, i2, i4, i8, f2, f4, f8, c4, c8, i1, bf},
          /* i2 */ {i2, i2, i2, i4, i8, f2, f4, f8, c4, c8, i2, bf},
          /* i4 */ {i4, i4, i4, i4, i8, f2, f4, f8, c4, c8, i4, bf},
          /* i8 */ {i8, i8, i8, i8, i8, f2, f4, f8, c4, c8, i8, bf},
          /* f2 */ {f2, f2, f2, f2, f2, f2, f4, f8, c4, c8, f2, f4},
          /* f4 */ {f4, f4, f4, f4, f4, f4, f4, f8, c4, c8, f4, f4},
          /* f8 */ {f8, f8, f8, f8, f8, f8, f8, f8, c8, c8, f8, f8},
          /* c4 */ {c4, c4, c4, c4, c4, c4, c4, c8, c4, c8, c4, c4},
          /* c8 */ {c8, c8, c8, c8, c8, c8, c8, c8, c8, c8, c8, c8},
          /* b1 */ {u1, i1, i2, i4, i8, f2, f4, f8, c4, c8, b1, bf},
          /* bf */ {bf, bf, bf, bf, bf, f4, f4, f8, c4, c8, bf, bf},
      };
  return _promoteTypesLookup[DataTypeToNum(x)][DataTypeToNum(y)];
}

```

这么做是不会漏掉任何一个数据类型的,而且还有一个优点,就是可以直接看到各个数据类型之间的提升关系.既符合人的思维,又符合机器的思维.<br>

Paddle 的 compare_op 在对于 int 的类型提升是存在问题的:<br>

![image-20241220194420005](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2916568339.webp)

问题出在这里:

```C++
    if ((is_support_float(x_dtype) && is_support_float(y_dtype)) ||
        (is_support_complex(x_dtype) || is_support_complex(y_dtype))) {
      return true;
    } else {
      PADDLE_THROW(common::errors::InvalidType(
          "Type promotion only support calculations between floating-point "
          "numbers and between complex and real numbers. But got different "
          "data type x: %s, y: %s.",
          x_dtype,
          y_dtype));
    }
```

这个逻辑是,如果两个都是 float,那么支持提升(看表是提升到精度高的 float 上),如果 x 或者 y 中有一个是 complex,那么统一提升到 complex 上.不管另一个是 bool,还是 int,还是 float.<br>

**缺了什么:** 缺少了 int 之间的类型提升,以及 int float之间的类型提升.

提升规则就是那个表格.<br>

我不知道为啥它不考虑这个,因为看到 int 和 float 的提升后都是 float,应该不会产生数据截断,也不会产生精度损失.<br>

int32 的取值范围是 -2147483648 到 2147483647,而 float32 的取值范围是 -3.4028235e+38 到 3.4028235e+38. 也不会因为数据溢出.<br>

对于其他 op 咱们不知道,但是对于 compare_op,那至少是有问题的.我就直接去改了.<br>

实际上也很简单只需要加上:<br>

```C++
if ((is_support_int(x_dtype) && is_support_int(y_dtype)) ||
    (is_support_int(x_dtype) && is_support_float(y_dtype))||
    (is_support_float(x_dtype) && is_support_int(y_dtype))) {
    return true;
}

```

![实例](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2916568338.webp)



solved,可以再补一下int->float,int->complex的单侧.
