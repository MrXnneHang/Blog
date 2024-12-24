# 【第四次正经 PR】 为 paddle.full 添加复数支持。 - CreateInferMeta

## 需求

目前我每次创建 complex Tensor 都相当麻烦。<br>

每次我都需要先创建两个 Float Tensor,然后再用`paddle.complex`把一个作为 real 一个作为 imag,如果能够支持一步到位，那么我们创建和测试效率也会高很多。<br>

这是 torch 的效果:<br>

```python
import torch

complex_tensor = torch.full((5, 5), 1+2j, dtype=torch.complex64)
print(complex_tensor)
tensor([[1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j],
        [1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j],
        [1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j],
        [1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j],
        [1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j, 1.+2.j]])
```

这是 paddle 目前的:<br>

```python
>>> complex_tensor = paddle.full(shape=[5, 5], fill_value=1+2j, dtype='complex64')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/xnne/miniconda3/envs/tests/lib/python3.9/site-packages/paddle/tensor/creation.py", line 1482, in full
    return fill_constant(shape=shape, dtype=dtype, value=fill_value, name=name)
  File "/home/xnne/miniconda3/envs/tests/lib/python3.9/site-packages/paddle/tensor/creation.py", line 1053, in fill_constant
    value = float(value)
TypeError: can't convert complex to float
```

接下来让我们一点点分析。

## infermeta 做了什么？

上一次我做了对 not_equal 和 equal 的复数类型支持:<br>

[【Paddle Tensor 第二期 复数类型支持问题 No.9、10】为 paddle.not_equal、paddle.equal 添加复数类型支持。](https://github.com/PaddlePaddle/Paddle/pull/69968)<br>

我上一次添加注册 complex 后没有对 infermeta 做任何修改。因为我也压根没看。<br>

```C++
void CompareRawInferMeta(const MetaTensor& x,
                         const MetaTensor& y,
                         int axis,
                         MetaTensor* out) {
  auto dim_x = x.dims();
  auto dim_y = y.dims();

  if (dim_x == dim_y) {
    out->share_meta(x);
  } else {
    int max_dim = std::max(dim_x.size(), dim_y.size());
    int axis = std::abs(dim_x.size() - dim_y.size());
    std::vector<int> x_dims_array(max_dim);
    std::vector<int> y_dims_array(max_dim);
    std::vector<int> out_dims_array(max_dim);
    funcs::GetBroadcastDimsArrays(dim_x,
                                  dim_y,
                                  x_dims_array.data(),
                                  y_dims_array.data(),
                                  out_dims_array.data(),
                                  max_dim,
                                  axis);

    out->set_dims(common::make_ddim(out_dims_array));
    out->share_lod(x);
  }
  if (!out->is_same_tensor(x)) {
    out->set_dtype(DataType::BOOL);
  }
}
```

这是它的 infermeta。<br>

它做了这些：<br>

- 检查形状是否相同，如果相同，metadata_tensor 直接使用 x 的形状。
- 如果不同，调用`GetBroadcastDimsArrays`函数，获取广播后的形状。
- 设置 metadata_tensor 的形状为广播后的形状。

这个 metadata_tensor 的作用我还没搞明白。<br>

但是它似乎和我们这个 compare 最终输出的形状是一样的。<br>

### full_kernel 的 InferMeta:

这一次我看到了 infermeta 有时候也单独对`dtype`进行处理。<br>

```C++
template <typename T, typename Context>
DenseTensor Full(const Context& dev_ctx, // 偏特化,for create
                 const IntArray& shape,
                 const Scalar& val) {
  DenseTensor dense_out;
  MetaTensor meta_out(&dense_out);
  DataType dtype = phi::CppTypeToDataType<T>::Type();
  CreateInferMeta(shape, dtype, &meta_out);
  FullKernel<T, Context>(dev_ctx, shape, val, dtype, &dense_out);
  return dense_out;
}
```

另外可以看到这个对于 infermeta 是在 kernel 中直接显式调用的。友好很多，上一次 compare_kernel 是一点都不友好的。<br>

### CreateInerMeta

这个意思不是`创建 InferMeta`，它的意思是`Create Tensor 的 kernel 的 InferMeta`。<br>

我注意到，对于`full`,`full_like`,等等和创建张量相关的函数,用的 InferMeata 都是这一个。<br>

```C++
void CreateInferMeta(const IntArray& shape,
                     DataType dtype,
                     MetaTensor* out,
                     MetaConfig config) {
  if (config.is_runtime || !shape.FromTensor()) {
    const auto& data = shape.GetData();
    for (size_t i = 0; i < data.size(); ++i) {
      PADDLE_ENFORCE_GE(
          data[i],
          0,
          common::errors::InvalidArgument(
              "Each value of attribute 'shape' is expected to be no less "
              "than 0. But received: shape[%u] = %d; shape = [%s].",
              i,
              data[i],
              common::make_ddim(data)));
    }
  }
  CreateInferMetaBase(shape.GetData(), dtype, DataLayout::NCHW, out);
}
```

它做了这些事情:<br>

- 检查 shape 是否合法。
- 调用`CreateInferMetaBase`函数，这个函数是真正的创建 infermeta 的函数。

### CreateInferMetaBase

```C++
void CreateInferMetaBase(const std::vector<int64_t>& shape,
                         DataType dtype,
                         DataLayout layout,
                         MetaTensor* out) {
  auto out_dims = common::make_ddim(shape);
  out->set_dims(out_dims);
  out->set_dtype(dtype);
  out->set_layout(layout);
}
```

直接继承传入的属性，没有进行任何检查和修改。<br>

所以我们支持 complex 也不需要对 infermeta 做任何修改。<br>

另外我也越发地感觉 infermeta 的 out 实际上就是我们最终的输出或者说输出的模板。<br>
