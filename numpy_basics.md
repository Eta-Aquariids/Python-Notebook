# NumPy 基础速查表

> 约定：`import numpy as np`
>
> 🏠 [返回笔记索引](README.md) ｜ 相关：[SciPy 速查表](scipy_basics.md) · [pandas 速查表](pandas_basics.md) · [Matplotlib 速查表](matplotlib_basics.md)

## 目录

- [安装与导入](#安装与导入)
- [数组创建](#数组创建)
- [数组属性](#数组属性)
- [数据类型](#数据类型)
- [索引与切片](#索引与切片)
- [形状操作](#形状操作)
- [拼接与分割](#拼接与分割)
- [运算与广播](#运算与广播)
- [聚合统计](#聚合统计)
- [数学函数](#数学函数)
- [线性代数](#线性代数)
- [排序与集合](#排序与集合)
- [随机数](#随机数)
- [读写文件](#读写文件)
- [视图 vs 拷贝](#视图-vs-拷贝)
- [常见坑与技巧](#常见坑与技巧)

---

## 安装与导入

```bash
pip install numpy            # 安装
pip install "numpy>=2.0"     # 指定版本
```

```python
import numpy as np

np.__version__     # 查看版本
np.info(np.array)  # 查看函数文档
```

---

## 数组创建

| 写法 | 说明 |
| --- | --- |
| `np.array([1, 2, 3])` | 从列表创建 |
| `np.zeros((2, 3))` | 全 0，形状 (2, 3) |
| `np.ones((2, 3))` | 全 1 |
| `np.full((2, 2), 7)` | 用 7 填充 |
| `np.empty((2, 2))` | 未初始化（快，但值随机） |
| `np.eye(3)` / `np.identity(3)` | 单位矩阵 |
| `np.arange(0, 10, 2)` | 左闭右开，步长 2 |
| `np.linspace(0, 1, 5)` | 0~1 均匀取 5 个点（含端点） |
| `np.zeros_like(a)` | 形状/类型同 `a` |
| `np.ones_like(a)` | 同上，全 1 |
| `np.full_like(a, 5)` | 同上，填充 5 |

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
print(a.shape)   # (2, 3)
print(a.dtype)   # int64

np.arange(6).reshape(2, 3)
# array([[0, 1, 2],
#        [3, 4, 5]])

np.linspace(0, 1, 5)   # array([0.  , 0.25, 0.5 , 0.75, 1.  ])
```

---

## 数组属性

```python
a = np.arange(24).reshape(2, 3, 4)

a.shape     # (2, 3, 4)   各维度长度
a.ndim      # 3           维度数
a.size      # 24          元素总数
a.dtype     # dtype('int64')
a.itemsize  # 8           单元素字节数
a.nbytes    # 192         总字节数 = size * itemsize
a.T         # 转置（沿所有轴反转）
a.base      # 若为视图则指向源数组
a.flags     # 内存布局信息（C/F 连续等）
```

---

## 数据类型

| 类型 | 说明 |
| --- | --- |
| `np.int8/16/32/64` | 有符号整数 |
| `np.uint8/16/32/64` | 无符号整数 |
| `np.float16/32/64` | 浮点（`float64` 为默认） |
| `np.bool_` | 布尔 |
| `np.complex64/128` | 复数 |
| `np.str_` | 字符串（定长） |
| `np.object_` | Python 对象 |

```python
a = np.array([1, 2, 3])
a.astype(np.float32)          # 转换类型
np.array([1, 2, 3], dtype=np.float64)

np.iinfo(np.int32)            # 整数范围信息
np.finfo(np.float32)          # 浮点精度信息
```

> ⚠️ 整型会静默溢出，浮点比较需用 `np.isclose` / `np.allclose`。

---

## 索引与切片

```python
a = np.arange(10)             # [0 1 2 3 4 5 6 7 8 9]

a[2]        # 3      正向索引
a[-1]       # 9      反向索引
a[2:5]      # [2 3 4]          切片，左闭右开
a[:5]       # [0 1 2 3 4]      省略起点
a[::2]      # [0 2 4 6 8]      步长
a[::-1]     # 反转

b = np.arange(12).reshape(3, 4)
b[1, 2]     # 6        行, 列
b[1]        # 第 1 行
b[:, 1]     # 第 1 列
b[0:2, 1:3] # 子矩阵
b[..., 0]   # 省略号：其他维度全取，最后一维取 0
```

**布尔索引**

```python
x = np.array([1, -2, 3, -4])

x[x > 0]              # [1 3]        过滤
x > 0                 # [True False True False]
np.where(x > 0, x, 0) # [1 0 3 0]    三元选择
np.where(x > 0)       # (array([0, 2]),)  返回下标
np.nonzero(x)         # 非零元素下标
x[(x > 0) & (x < 3)]  # 组合条件要用 & | ~，不能用 and/or/not
```

**花式索引**

```python
y = np.array([10, 20, 30, 40, 50])

y[[0, 2, 4]]        # [10 30 50]   按下标列表取值
y[np.array([0, 2])] # 同上
y[[True, False, True, False, True]]

m = np.arange(12).reshape(3, 4)
m[[0, 2]]           # 取第 0、2 行
m[[0, 2], [1, 3]]   # 取 (0,1) 和 (2,3)
```

---

## 形状操作

| 方法 | 说明 |
| --- | --- |
| `a.reshape(2, 3)` | 改变形状（视图） |
| `a.reshape(-1, 2)` | `-1` 表示自动推导 |
| `a.ravel()` | 展平，返回视图（尽量） |
| `a.flatten()` | 展平，返回拷贝 |
| `a.T` / `a.transpose()` | 转置 |
| `a.swapaxes(0, 1)` | 交换两个轴 |
| `a.squeeze()` | 去掉长度为 1 的轴 |
| `np.expand_dims(a, 0)` | 在指定位置插入长度为 1 的轴 |
| `a.resize(...)` | 原地改变形状（会填充/截断） |
| `np.resize(a, ...)` | 返回新数组（循环重复元素） |

```python
a = np.arange(6)
a.reshape(2, 3)          # 2 行 3 列
a.reshape(3, -1)         # 3 行，列数自动 = 2
a.reshape(-1)            # 展平
a[:, None].shape         # (6, 1)  用 None 增加轴
a[:, np.newaxis].shape   # 同上
```

---

## 拼接与分割

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])            # [1 2 3 4 5 6]
np.vstack([a, b])                 # 竖向堆叠 -> (2, 3)
np.hstack([a, b])                 # 横向拼接 -> (6,)
np.stack([a, b])                  # 新增轴 -> (2, 3)
np.stack([a, b], axis=1)          # -> (3, 2)
np.column_stack([a, b])           # 按列拼

np.split(np.arange(6), 3)         # 均分为 3 份
np.array_split(np.arange(7), 3)   # 不均分也允许
np.hsplit(arr, 2)                 # 横向分割
np.vsplit(arr, 2)                 # 竖向分割
```

> `concatenate` 沿已有轴拼接；`stack` 会新建一个轴。所有数组除拼接轴外形状必须一致。

---

## 运算与广播

```python
a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

a + b, a - b, a * b, a / b        # 逐元素运算
a ** 2                            # 平方
a % 2, a // 2                     # 取余、整除
-a                                # 取负
a + 1                             # 标量广播

np.add(a, b)                      # 等价函数形式
a += 1                            # 原地修改
a @ b                             # 矩阵乘法（1-D 时为点积）等于 np.dot(a, b)
```

**比较与逻辑**

```python
a > 2                  # 逐元素布尔数组
np.array_equal(a, b)   # 数组是否完全相同
np.allclose(a, b)      # 浮点近似相等
np.logical_and(a > 0, a < 3)
np.any(a > 2)          # 是否存在
np.all(a > 2)          # 是否全部满足
```

**广播规则**（从右对齐维度，逐维比较）

1. 维度相等，或
2. 其中一维为 1（可被拉伸），或
3. 缺失的维度视为 1

```python
(3, 1) + (1, 4)   # -> (3, 4)
(5, 3) + (3,)     # -> (5, 3)
(2, 3) + (4,)     # ❌ 报错：3 != 4
```

---

## 聚合统计

```python
a = np.array([[1, 2, 3], [4, 5, 6]])

a.sum()            # 21        全部求和
a.sum(axis=0)      # [5 7 9]   按列（垮掉第 0 轴）
a.sum(axis=1)      # [6 15]    按行
a.mean()           # 3.5
a.mean(axis=1)     # [2. 5.]
a.std()            # 标准差
a.var()            # 方差
a.min(), a.max()   # 最小 / 最大
a.argmin(), a.argmax()   # 最小 / 最大值下标
a.cumsum()         # 累加
a.cumprod()        # 累乘

np.median(a)       # 中位数
np.percentile(a, 50)   # 百分位数
np.quantile(a, 0.5)    # 分位数
np.ptp(a)          # 极差 = max - min
```

| 参数 | 作用 |
| --- | --- |
| `axis=0` | 沿第 0 轴压缩（对每一列运算） |
| `axis=1` | 沿第 1 轴压缩（对每一行运算） |
| `keepdims=True` | 保留被压缩的轴（长度为 1），便于广播 |

> `np.nanmean` / `np.nansum` / `np.nanstd` 会忽略 `NaN`。

---

## 数学函数

```python
x = np.array([1, 4, 9])

np.sqrt(x)          # [1. 2. 3.]   平方根
np.square(x)        # 平方
np.exp(x)           # e^x
np.log(x)           # ln
np.log2(x)          # log2
np.log10(x)         # log10
np.log1p(x)         # ln(1+x)，小值更精确
np.abs(x)           # 绝对值
np.power(x, 3)      # 幂
np.sin(x), np.cos(x), np.tan(x)
np.deg2rad(180), np.rad2deg(np.pi)

np.floor(x)         # 向下取整
np.ceil(x)          # 向上取整
np.round(x, 2)      # 四舍五入保留 2 位
np.trunc(x)         # 截断小数
np.clip(x, 2, 8)    # 限制到 [2, 8]
np.sign(x)          # 符号

np.maximum(a, b)    # 逐元素取大
np.minimum(a, b)    # 逐元素取小
np.sum / np.prod / np.mean / np.std  # 函数形式（不带 axis 时展平后运算）
```

---

## 线性代数

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

A @ B                # 矩阵乘法
np.matmul(A, B)      # 同上
np.dot(A, B)         # 矩阵乘法（1-D 时为点积）

A.T                  # 转置
np.linalg.inv(A)     # 逆矩阵
np.linalg.det(A)     # 行列式
np.linalg.matrix_rank(A)   # 秩
np.linalg.norm(A)          # Frobenius 范数
np.trace(A)                # 迹

np.linalg.solve(A, b)      # 解 Ax = b（比先求逆更快更稳）
np.linalg.lstsq(A, b, rcond=None)   # 最小二乘
eigvals, eigvecs = np.linalg.eig(A) # 特征值 / 特征向量
U, S, Vt = np.linalg.svd(A)         # 奇异值分解
L = np.linalg.cholesky(np.eye(2))   # Cholesky 分解
```

---

## 排序与集合

```python
a = np.array([3, 1, 2])

np.sort(a)            # [1 2 3]    返回副本
a.sort()              # 原地排序
np.argsort(a)         # [1 2 0]    排序后原下标
np.argmax(a), np.argmin(a)

b = np.array([1, 2, 2, 3])
np.unique(b)                   # [1 2 3]
np.unique(b, return_counts=True)   # (array([1,2,3]), array([1,2,1]))

np.intersect1d(b, [2, 3, 4])   # 交集
np.union1d(b, [4])             # 并集
np.setdiff1d(b, [2])           # 差集
np.isin(b, [2, 3])             # 元素是否在集合中

np.searchsorted(np.array([1, 3, 5]), [2, 4])   # 插入位置下标
```

> `axis` 对二维数组同样适用，如 `np.sort(a, axis=1)`。

---

## 随机数

**推荐（新式 Generator 接口）**

```python
rng = np.random.default_rng(seed=42)      # seed 固定则结果可复现

rng.random((2, 3))          # [0, 1) 均匀分布
rng.integers(0, 10, size=5) # [0, 10) 整数
rng.normal(0, 1, size=1000) # 正态分布 均值 0 标准差 1
rng.uniform(0, 1, size=5)   # 均匀分布
rng.choice([1, 2, 3], size=2, replace=False)   # 无放回抽样
rng.shuffle(arr)            # 原地打乱
rng.permutation(arr)        # 返回打乱后的副本
rng.binomial(10, 0.5, size=5)
rng.poisson(3, size=5)
```

**旧式接口（仍可用，但不建议新代码）**

```python
np.random.seed(42)
np.random.rand(2, 3)        # [0,1) 均匀
np.random.randn(2, 3)       # 标准正态
np.random.randint(0, 10, size=5)
np.random.normal(loc=0, scale=1, size=5)
```

---

## 读写文件

```python
a = np.arange(10)

# 二进制（.npy）—— 快，保留 dtype 和 shape
np.save("a.npy", a)
a = np.load("a.npy")

# 多个数组打包（.npz）
np.savez("data.npz", x=a, y=a * 2)
d = np.load("data.npz")
d["x"], d["y"]

# 文本（CSV / TXT）
np.savetxt("a.csv", a, delimiter=",", fmt="%.2f")
np.loadtxt("a.csv", delimiter=",")
np.loadtxt("a.csv", delimiter=",", skiprows=1, usecols=(0, 1))  # 跳过表头、选列

# Pandas 对复杂表格更合适
# pd.read_csv("a.csv")
```

---

## 视图 vs 拷贝

| | 视图 (view) | 拷贝 (copy) |
| --- | --- | --- |
| 共享内存 | ✅ | ❌ |
| 典型来源 | 切片、`reshape`、`ravel`、`T` | 花式索引、`flatten`、`copy()` |
| 修改影响原数组 | ✅ | ❌ |

```python
a = np.arange(5)
v = a[1:4]        # 视图
v[0] = 99         # a 也随之改变！

c = a[1:4].copy() # 显式拷贝，安全
c[0] = 0          # a 不变

a.reshape(5, 1).base is a   # 不为 None -> 是视图
```

---

## 常见坑与技巧

**1. 列表与数组的运算差异**

```python
[1, 2] * 2      # [1, 2, 1, 2]   列表重复
np.array([1, 2]) * 2   # [2 4]    逐元素乘法
```

**2. 逻辑运算用位运算符**

```python
x[(x > 0) & (x < 5)]   # ✅
x[(x > 0) and (x < 5)] # ❌ ValueError / 语义错误
```

**3. 默认整数除法在 Python 3 中已是真除法**

```python
np.array([1, 2]) / 2   # [0.5 1. ]
```

**4. `axis` 的含义：被压扁的轴**

```python
a.cumsum(axis=0)   # 沿行方向累积（每列自上而下累加）
```

**5. 性能提示**

- 用向量化操作替代 `for` 循环
- 尽量使用原地操作（`a += 1`、`out=` 参数）减少内存分配
- 预分配结果数组：`out = np.empty((n, k))`
- 沿 C 连续数组的最后一轴访问最快
- 大数组避免不必要的 `copy()` 和 `flatten()`

**6. 内存布局与视图安全**

```python
np.ascontiguousarray(a)    # 保证 C 连续
a.flags["C_CONTIGUOUS"]    # 检查是否 C 连续
```

**7. 打印控制**

```python
np.set_printoptions(precision=3, suppress=True, linewidth=120)
# suppress=True 抑制科学计数法，适合小数值
```

---

## 参考

- 官方文档：https://numpy.org/doc/stable/
- 官方速查（PDF）：https://numpy.org/doc/stable/user/absolute_beginners.html
- NumPy 2.0 迁移指南：https://numpy.org/doc/stable/numpy_2_0_migration_guide.html
