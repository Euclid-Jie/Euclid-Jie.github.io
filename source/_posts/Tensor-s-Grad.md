---
title: Tensor's Grad
date: 2023-12-21 23:14:34
tags: [python, torch]
---
### 何为梯度 `grad`

记录`loss`对参数的偏导，用于更新参数，模型训练的时候，默认为参数创建，可以使用`tensor.requires_grad`查询张量是否带梯度

```python
import torch
tensor_demo = torch.tensor([[1,2,3],[4,5,6]])
# 返回False说明不带grad
tensor_demo.requires_grad

# 创建一个带有grad的张量
# 注意，int不能带有grad，故使用float32
tensor_demo = torch.tensor(
    [[1, 2, 3], [4, 5, 6]], dtype=torch.float32, requires_grad=True
)
# 返回True说明这是一个带grad的张量
tensor_demo.requires_grad
```

### `grad`有什么用

可以利用`torch`的自动求导特性，得到`loss`关于次参数的导数，并利用此导数进行参数更新

具体而言，模型的一个`epoch`有这么五步

> 如果模型的设计是，参数是每个`step`更新，那每个`step`都有这么几步

1. 向前传播，`forward`，即使用模型，传入`feature`，得到模型输出`pred`

    ```python
    pred = model(feature)
    ```

2. 计算`loss`，这里的`loss_func`为自己定义的函数，比如常见的`L1 loss`

   ```python
   loss = loss_func(pred, target)
   ```

3. 向后传播，`backward`，即对模型各个参数计算对应的`grad`

    ```python
    loss.backward()
    ```

4. 更新模型参数，根据自己定义的优化器`optimizer`策略，进行参数更新

   ```python
   optimizer.step() 
   ```

5. 将模型所有参数的`grad`重置为0

   ```python
   optimizer.zero_grad()
   ```

### 清空单个张量的`grad`

当张量被初始化时，是没有梯度，是不用清空的，此时`grad`为`None`

```python
import torch
tensor_demo = torch.tensor([[1,2,3],[4,5,6]],dtype=torch.float32).requires_grad_(True)
# 啥也不返回说明grad是None
tensor_demo.grad
```

一旦基于此`tensor`的计算的值进行了`backward`，也就是依据计算图计算了各个参数的grad，此`tensor`附带的`grad`就不是`None`了

```python
import torch
tensor_demo = torch.tensor([[1,2,3],[4,5,6]],dtype=torch.float32).requires_grad_(True)
# 基于tesor_demo计算得到res
res =(tensor_demo**2).sigmoid().sum()
# 计算grad
res.backward()
# 打印tensor的grad
tensor_demo.grad
```

梯度在每次`backward`是不会被覆盖的，而是累计，所以需要手动清空，这里的清空是指将梯度设置为0；需要注意的是如果`grad`是`None`，是不能被清空的

```python
if tensor_demo.grad is not None:
    print("grad is not None, will set grad to zeros")
    tensor_demo.grad.zero_()
# 如果grad不是None，将被重置为0
tensor_demo.grad
```

### 将`grad` 转为 `feature importance`

对`feature`添加`grad`，进行`backward`后，基于`grad`计算`feature importance`

1. 读取模型参数，使用模型预测

   ```python
   import torch
   
   model = DailyNet.load_from_checkpoint(
       Path("epoch04.ckpt"),
   )
   # 如果模型中有dropout等层，需要进行处理，否则后续操作有随机性
   model._gru_block._gru.dropout = 0.0
   
   # 可以将参数的梯度去掉, 并不影响featuer的grad计算
   # for param in model.parameters():
   #     param.detach_()
   
   # NOTE 需要给feature添加grad
   input_feature = input_feature.requires_grad_(True)
   preds = model(input_feature).cpu()
   ```

2. 计算`loss`，并`backward`

   > `loss_func`为自定义的损失函数，`target`为真实值

   ```python
   if input_feature.requires_grad:
       if input_feature.grad is not None:
           input_feature.grad.zero_()
   else:
       raise KeyError("input_feature requires grad")
   
   loss = loss_func(preds, target)
   
   loss.backward()
   ```

3. 根据`grad`计算`feature importance`

   ```python
   grad = input_feature.grad
   grad.abs().sum(axis=(0, 1)) / grad.abs().sum()
   ```
