---
title: PCA impl with python
tags:
  - python
  - 
categories:
  - Statistics
date: 2023-12-22 22:33:06
---

对于送入网络的`feature`，希望通过`pca`降维的方法，尽可能保留信息的情况下，减少模型参数，以缩短训练和推理时间，甚至有可能减少数据中的噪声？

但`pca`为纯线性变换，且没有考虑对目标的优化，进一步可以使用`vae`方法

## 主成分分析法 `PCA`

## 理论

简单来说就是把高纬数据，投影到低维，不过肯定会有信息损失，一般保留80%的信息，或者指定保留多少个主成分。

具体的操作老师讲过证明，但是具体推导和证明我不记得了，反正就是对原始矩阵（假设大小为`(N,T)`）进行特征分解，得到特征值和特征向量，选取特征值大的对应的特征向量（大小为`(N,t)`)作为基，矩阵相乘（结果的大小为`(N,t)`）即得到`t`个主成分，而这个`(N,t)`大小的矩阵，即作为压缩后的特征

## `python` 实现

使用`python`实现，这里直接调用`sklearn`中的包实现，只不过记录两种方式，一种是全量，一种是增量。

### 全量方式

直接读取所有数据，然后喂给`PCA`，即完成主成分分析，需要注意的点有

- 原始数据的`shape`为（数据点数量，特征个数），一般来说数据点数量 >= 特征个数
- 无需对原始数据做标准化，`sklearn`会帮你做
- 创建`pca`时候，需要指定预期保留的主成分个数，要小于特征个数
- 调用`fit_transform`后返回的是主成分，也就是压缩后的特征
- `explained_variance_sum`为主成分能够解释的原始数据中的变差占比
- `components`的为拟合出来的一组基，再拿到`(N,T)`的数据，直接可以计算得到主成分，$(N,T) * (T,t)  \to (N,t)$

```python
import numpy as np
from sklearn.decomposition import PCA
# shape = (N, T)
data = np.random.rand(100, 10)

# Step 1: Create an instance of PCA
pca = PCA(n_components=8)

# Step 2: Fit the data and transform it
reduced_data = pca.fit_transform(data)

# Step 3: calculate the explained variance
explained_variance = pca.explained_variance_ratio_
explained_variance_sum = explained_variance.sum()
print("explained_variance_sum", explained_variance_sum)

# Step 4: Get the components
# shape = (t, T)
components = pca.components_
```

### 增量方式

有时候数据量很大，需要分块拟合，也可能是`dataloader`的形式，需要一次又一次喂给`pca`，这时候需要增量方式，官方文档[sklearn.decomposition.IncrementalPCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.IncrementalPCA.html)

如果是挨个喂，这样写

> 这里只是`fit`，目的是获取基，挨个喂使用的是`partial_fit`，而非`fit`

```python
from sklearn.decomposition import IncrementalPCA

# Step 1: Create an instance of PCA
ipca = IncrementalPCA(n_components=100)

# Step 2: Fit the data step by step
for feature_batch in feature_list:
    ipca.partial_fit(feature_batch)
```

如果是希望它自己分块

> 这里只是`fit`，目的是获取基

```python
from sklearn.decomposition import IncrementalPCA

# Step 1: Create an instance of PCA
ipca = IncrementalPCA(n_components=100,batch_size=10)

# Step 2: Fit the data step by step
ipca.fit(all_feature)
```

最后获得主成分、已解释变差、基的方式是一样的

```python
# 主成分
ipca.fit_transform(data)
# 已解释变差
ipca.explained_variance_ratio_.sum()
# 基
ipca.components_
```

