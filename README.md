# Python Notebook

个人 Python 笔记与速查表集合，以纯 Markdown 维护，便于查阅、检索和 git diff。

## 目录

| 笔记 | 内容概要 | 主要依赖 |
| --- | --- | --- |
| [NumPy 基础速查表](numpy_basics.md) | 数组创建、dtype、索引切片、形状操作、广播、聚合统计、线性代数、随机数、文件读写 | `numpy` |
| [SciPy 基础速查表](scipy_basics.md) | 常量与特殊函数、优化、插值、积分与 ODE、稀疏矩阵、信号处理、统计检验、空间算法、图像处理 | `scipy` |
| [pandas 基础速查表](pandas_basics.md) | Series/DataFrame、选择与筛选、缺失值、分组聚合、合并重塑、字符串、时间序列、性能技巧 | `pandas` |
| [Matplotlib 基础速查表](matplotlib_basics.md) | 绘图类型、颜色线型、坐标轴与刻度、图例标注、中文与字体、子图布局、2D/3D 可视化、样式导出、动画 | `matplotlib` |

## 使用

```bash
# 安装全部依赖
pip install numpy scipy pandas matplotlib

# 或按需安装
pip install numpy

# 可视化含动画导出等可选依赖
pip install "matplotlib[all]"
```

笔记中的示例均为可直接运行的代码片段，默认已执行过下列导入：

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import optimize, stats, linalg, sparse, signal, interpolate, integrate
```

## 约定

- 以 `#` 开头的表格行或注释表示说明，非可执行代码。
- ⚠️ 标记处为容易踩坑或已废弃的写法。
- 示例优先使用各库当前推荐的 API（如 NumPy 的 `default_rng`、pandas 的 `agg` 命名聚合、Matplotlib 的面向对象 `fig, ax` 风格）。

## 相关链接

- NumPy 官方文档：https://numpy.org/doc/stable/
- SciPy 官方文档：https://docs.scipy.org/doc/scipy/
- pandas 官方文档：https://pandas.pydata.org/docs/
- Matplotlib 官方文档：https://matplotlib.org/stable/
- Matplotlib 速查手册：https://matplotlib.org/cheatsheets/

## 许可

个人学习笔记，内容可自由参考使用。
