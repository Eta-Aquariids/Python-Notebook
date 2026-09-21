# pandas 基础速查表

> 约定：`import pandas as pd`，`import numpy as np`
> 示例基于 pandas 2.x（含 3.0 的 Copy-on-Write 行为）
>
> 🏠 [返回笔记索引](README.md) ｜ 相关：[NumPy 速查表](numpy_basics.md) · [SciPy 速查表](scipy_basics.md)

## 目录

- [安装与数据结构](#安装与数据结构)
- [创建 DataFrame](#创建-dataframe)
- [读写数据](#读写数据)
- [查看与摘要](#查看与摘要)
- [选择与筛选](#选择与筛选)
- [索引操作](#索引操作)
- [缺失值处理](#缺失值处理)
- [增删改与类型转换](#增删改与类型转换)
- [apply / map 系列](#apply--map-系列)
- [排序与排名](#排序与排名)
- [分组聚合 groupby](#分组聚合-groupby)
- [透视表与交叉表](#透视表与交叉表)
- [合并与连接](#合并与连接)
- [重塑与变形](#重塑与变形)
- [字符串处理 str](#字符串处理-str)
- [分类类型 category](#分类类型-category)
- [时间序列](#时间序列)
- [窗口计算](#窗口计算)
- [性能技巧](#性能技巧)
- [输出数据](#输出数据)
- [常见坑](#常见坑)

---

## 安装与数据结构

```bash
pip install pandas
pip install pyarrow        # 推荐：Arrow 后端、parquet/feather 读写
```

```python
import pandas as pd
pd.__version__

pd.set_option("display.max_columns", None)   # 显示全部列
pd.set_option("display.max_rows", 100)
pd.set_option("display.width", 200)
pd.set_option("display.precision", 3)
```

| 结构 | 说明 | 轴 |
| --- | --- | --- |
| `Series` | 带标签的一维数组，有 `index` + 一列 `values` | 1 |
| `DataFrame` | 二维表格，`index`（行标签）+ `columns`（列标签） | 2 |
| `Index` | 不可变的轴标签容器，可含多级 | — |

```python
s = pd.Series([1, 2, 3], index=["a", "b", "c"], name="值")
s.values         # array([1, 2, 3])
s.index          # Index(['a', 'b', 'c'])
s.dtype          # int64
s.to_frame()     # Series → 单列 DataFrame
```

---

## 创建 DataFrame

```python
# 从字典（最常用）
df = pd.DataFrame({
    "name": ["Alice", "Bob", "Cindy"],
    "age": [25, 30, 35],
    "city": ["BJ", "SH", "GZ"],
})

# 从记录列表
pd.DataFrame([{"a": 1, "b": 2}, {"a": 3, "b": 4}])

# 从 NumPy 数组
pd.DataFrame(np.arange(6).reshape(3, 2), columns=["x", "y"])

# 指定索引与列顺序
pd.DataFrame(data, index=["r1", "r2"], columns=["a", "b"])

# 常用构造函数
pd.Series([1, 2, 3])
pd.date_range("2026-01-01", periods=5, freq="D")   # DatetimeIndex
pd.timedelta_range("1 days", periods=3)
pd.Categorical(["a", "b", "a"])
pd.MultiIndex.from_product([["A", "B"], [1, 2]], names=["grp", "id"])
```

---

## 读写数据

```python
# 读取
pd.read_csv("data.csv", sep=",", encoding="utf-8")
pd.read_csv("data.csv", header=None, names=["a", "b"])       # 无表头
pd.read_csv("data.csv", index_col=0, usecols=["a", "b"])
pd.read_csv("data.csv", dtype={"id": "int32", "code": "category"})
pd.read_csv("data.csv", parse_dates=["date"], na_values=["NA", "-"])
pd.read_csv("data.csv", skiprows=3, nrows=1000, comment="#")
pd.read_csv("big.csv", chunksize=100_000)                    # 返回迭代器，分块处理
pd.read_excel("data.xlsx", sheet_name="Sheet1")
pd.read_json("data.json", orient="records", lines=True)      # JSON Lines
pd.read_parquet("data.parquet")                              # 列式，最快最省内存
pd.read_sql("SELECT * FROM t", con=engine)
pd.read_clipboard()
pd.read_html("https://example.com/table")                    # 返回 DataFrame 列表

# 写出
df.to_csv("out.csv", index=False, encoding="utf-8-sig")      # Excel 友好
df.to_excel("out.xlsx", sheet_name="结果", index=False)
df.to_parquet("out.parquet", compression="snappy")
df.to_json("out.json", orient="records", force_ascii=False)
df.to_sql("table", con=engine, if_exists="replace", index=False)
df.to_markdown()                                             # 生成 Markdown 表格
```

| 格式 | 优点 | 场景 |
| --- | --- | --- |
| CSV | 通用、可读 | 交换、小数据 |
| Parquet | 快、压缩率高、保留 dtype | 中间结果、大数据 |
| Feather | 极快 | 进程间传递 |
| Excel | 业务交付 | 报表 |

> ⚠️ `encoding="utf-8-sig"` 可避免 Excel 打开中文 CSV 乱码。

---

## 查看与摘要

```python
df.head(10)              # 前 10 行（默认 5）
df.tail(5)               # 后 5 行
df.sample(5, random_state=0)   # 随机抽样
df.info()                # 列名、非空数、dtype、内存占用
df.describe()            # 数值列统计
df.describe(include="all")     # 含对象/分类列
df.shape                 # (行数, 列数)
df.columns.tolist()      # 列名列表
df.dtypes                # 每列类型
df.index                 # 行索引
df.T                     # 转置
df.values                # → ndarray
df.to_numpy()            # 同上（推荐）

df["city"].value_counts()             # 频次统计（降序）
df["city"].value_counts(normalize=True)   # 占比
df["city"].nunique()                  # 唯一值个数
df["city"].unique()                   # 唯一值数组
df["age"].mean(), df["age"].std()     # 单列聚合

df.memory_usage(deep=True)            # 内存占用（deep 统计字符串真实大小）
df.isna().sum()                       # 每列缺失值数量
df.duplicated().sum()                 # 重复行数量
df.corr(numeric_only=True)            # 相关系数矩阵
df.cov(numeric_only=True)
df.quantile([0.25, 0.5, 0.75])
```

---

## 选择与筛选

**列选择**

```python
df["age"]                 # Series
df[["name", "age"]]       # DataFrame（注意双层方括号）
df.age                    # 属性访问，仅当列名是合法标识符且不与方法重名
df.filter(like="a")       # 列名包含 "a"
df.filter(regex="^a")     # 正则匹配列名
df.filter(items=["name"])
df.select_dtypes(include="number")      # 按类型选列
df.select_dtypes(exclude="object")
```

**行/元素定位 `.loc` 与 `.iloc`**（首选，语义明确）

| 写法 | 依据 | 端点 |
| --- | --- | --- |
| `df.loc[...]` | 标签 | **包含**右端点 |
| `df.iloc[...]` | 整数位置 | 不包含右端点 |
| `df.at[r, c]` | 标签，单个元素 | 最快 |
| `df.iat[i, j]` | 位置，单个元素 | 最快 |

```python
df.loc[0]                          # 索引标签为 0 的行
df.loc[:, "age"]                   # 所有行的 age 列
df.loc[0:2, "name":"age"]          # 行标签 0~2（含），列同理
df.loc[df["age"] > 28, ["name"]]   # 布尔 + 列筛选
df.loc[[0, 2], ["name", "age"]]    # 标签列表

df.iloc[0]                         # 第 0 行
df.iloc[-1]                        # 最后一行
df.iloc[0:2, 1:3]                  # 第 0~1 行，第 1~2 列
df.iloc[:, [0, -1]]

df.at[0, "name"]                   # 单元素，标签
df.iat[0, 1]                       # 单元素，位置
```

**布尔索引**

```python
df[df["age"] > 28]
df[(df["age"] > 25) & (df["city"] == "BJ")]     # 必须用 & | ~，条件加括号
df[df["city"].isin(["BJ", "GZ"])]
df[df["age"].between(26, 34)]
df[~df["city"].isin(["BJ"])]                      # 取反
df[df["name"].str.startswith("A")]
df[df["age"].notna()]
```

**`query` / `eval`**（可读性好，大表更快）

```python
df.query("age > 28 and city == 'BJ'")
df.query("city in ['BJ', 'GZ']")
df.query("age > @threshold")             # @ 引用外部变量
df.query("name.str.startswith('A')")
df.eval("total = age * 2")               # 添加计算列
```

**`where` / `mask`**（条件替换，保持形状）

```python
df.where(df["age"] > 28, 0)              # 不满足条件 → 0
df["age"].mask(df["age"] > 28, 0)        # 满足条件 → 0（与 where 相反）
df["age"].clip(lower=0, upper=100)       # 截断
```

---

## 索引操作

```python
df.set_index("name")                     # 用列作索引
df.set_index("name", drop=False)         # 保留该列
df.reset_index()                         # 索引还原为列
df.reset_index(drop=True)                # 直接丢弃，不生成列
df.reindex([0, 1, 5])                    # 按新标签重新对齐，缺失填 NaN
df.reindex(columns=["name", "age"])
df.sort_index()                          # 按索引排序
df.sort_index(axis=1, ascending=False)   # 按列名排序
df.rename(columns={"age": "年龄"})
df.rename(columns=str.lower)
df.rename_axis("行", axis=0).rename_axis("列", axis=1)
df.index.name = "id"

# MultiIndex（多级索引）
mi = df.set_index(["city", "name"])
mi.loc["BJ"]                             # 第一级切片
mi.loc[("BJ", "Alice")]                  # 精确定位
mi.xs("BJ", level="city")                # 跨级取值
mi.droplevel("city")
mi.reset_index()
pd.MultiIndex.from_tuples([("A", 1), ("B", 2)], names=["g", "i"])
```

**`Series.map` 与 `Index` 对齐特性**

```python
s = pd.Series([1, 2, 3], index=["a", "b", "c"])
t = pd.Series([10, 20, 30], index=["b", "c", "d"])
s + t        # 按索引对齐，不重叠处为 NaN
s.add(t, fill_value=0)   # 缺失按 0 处理
```

---

## 缺失值处理

pandas 中缺失值表示为 `np.nan`（数值）、`None`、`pd.NaT`（时间）。

```python
df.isna(), df.notna()          # 布尔矩阵
df.isna().sum()                # 每列缺失数
df.isna().any(axis=1)          # 每行是否有缺失

df.dropna()                    # 删除含缺失的行
df.dropna(axis=1)              # 删除含缺失的列
df.dropna(how="all")           # 全为缺失才删
df.dropna(thresh=2)            # 至少保留 2 个非缺失值
df.dropna(subset=["age"])      # 只看指定列

df.fillna(0)                   # 常量填充
df.fillna({"age": 0, "city": "未知"})   # 分列填充
df["age"].fillna(df["age"].mean())
df.ffill()                     # 用前一个有效值填充（原 pad）
df.bfill()                     # 用后一个有效值填充（原 backfill）
df.ffill(limit=1)              # 最多连续填 1 个
df["age"].interpolate(method="linear")      # 线性插值
df["age"].interpolate(method="time", limit_direction="both")
df["age"].replace([-1, -999], np.nan)       # 把占位符变成 NaN

# 缺失值检测工具
df.isna().sum().pipe(lambda s: s[s > 0])
```

> ⚠️ `fillna(method="ffill")` 已废弃，请直接用 `ffill()` / `bfill()`。

---

## 增删改与类型转换

```python
# 新增列
df["total"] = df["age"] * 2
df.assign(total=lambda d: d["age"] * 2,          # 可链式，支持引用前面新建的列
          flag=lambda d: d["total"] > 50)

# 插入 / 删除
df.insert(1, "new", [1, 2, 3])
df.drop(columns=["total"])
df.drop(index=[0])
df.drop(columns=["total"], inplace=False)        # inplace 已不推荐

# 重命名
df.rename(columns={"age": "年龄"})

# 类型转换
df["age"].astype("float32")
df["age"].astype(int)
df.astype({"age": "int32", "name": "string"})
pd.to_numeric(df["age"], errors="coerce")        # 转数字，失败置 NaN
df.convert_dtypes()                              # 自动选更合适的 dtype
df.convert_dtypes(dtype_backend="pyarrow")       # Arrow 后端

# 逐元素替换
df["city"].replace({"BJ": "北京", "SH": "上海"})
df["city"].replace("BJ", "北京")

# 增加/删除行
pd.concat([df, pd.DataFrame([{"name": "Dave", "age": 40}])], ignore_index=True)
df.drop_duplicates(subset=["name"], keep="first")
```

---

## apply / map 系列

| 方法 | 作用对象 | 粒度 |
| --- | --- | --- |
| `Series.map(f)` | Series | 逐元素 |
| `Series.apply(f)` | Series | 逐元素 |
| `DataFrame.map(f)` | DataFrame | 逐元素（2.1+，替代 `applymap`） |
| `DataFrame.apply(f, axis=0)` | DataFrame | 逐列 |
| `DataFrame.apply(f, axis=1)` | DataFrame | 逐行 |
| `Series.map(dict)` | Series | 按字典映射 |

```python
df["name"].str.upper()
df["age"].map(lambda x: x + 1)
df["city"].map({"BJ": "北京", "SH": "上海"})       # 字典映射，未匹配为 NaN
df.map(lambda x: str(x).strip())                  # 全表逐元素

df.apply(np.sum, axis=0)                          # 每列求和
df.apply(lambda row: row["age"] * 2, axis=1)      # 逐行（慢，尽量向量化）
df.groupby("city")["age"].transform("mean")       # 广播回原形状

df.pipe(func)                                     # 把 df 作为整体传给函数，便于链式
```

> ⚠️ `axis=1` 的 `apply` 是 Python 级循环，大表极慢；优先用向量化或 `np.where`。

---

## 排序与排名

```python
df.sort_values("age")                             # 升序
df.sort_values("age", ascending=False)
df.sort_values(["city", "age"], ascending=[True, False])
df.sort_values("age", na_position="first")
df.sort_index()

df["age"].rank(method="min")                      # 排名，method: average/min/max/dense/first
df.nlargest(3, "age")
df.nsmallest(2, "age")
df["city"].value_counts().idxmax()                # 出现最多的类别
```

---

## 分组聚合 groupby

```python
g = df.groupby("city")
g.size()                       # 每组行数
g["age"].mean()                # 分组求均值
g[["age"]].agg(["mean", "std", "count"])
g.agg({"age": ["mean", "max"], "name": "count"})   # 分列不同聚合

# 命名聚合（推荐，输出列名清晰）
df.groupby("city").agg(
    avg_age=("age", "mean"),
    max_age=("age", "max"),
    n=("name", "count"),
)

# 常用参数
df.groupby("city", as_index=False)["age"].mean()   # 结果保持扁平表
df.groupby("city", dropna=False).size()            # 保留 NaN 分组
df.groupby("city", observed=True).size()           # 分类列只保留出现的组合
df.groupby(["city", "name"]).sum()                 # 多列分组（结果 MultiIndex）
df.groupby(level=0).sum()                          # 按索引级别分组
df.groupby(lambda i: i % 2).sum()                  # 按函数分组

# transform：返回与原表对齐的结果，便于新增列
df["city_mean"] = df.groupby("city")["age"].transform("mean")
df["rank_in_city"] = df.groupby("city")["age"].rank()

# filter：按组条件筛选
df.groupby("city").filter(lambda x: len(x) > 1)

# apply：对每组做任意操作
df.groupby("city").apply(lambda x: x.nlargest(1, "age"), include_groups=False)

# 遍历
for key, sub in df.groupby("city"):
    print(key, len(sub))

# 其他
df.groupby("city")["age"].cumsum()      # 组内累加
df.groupby("city")["age"].shift(1)      # 组内上移
df.groupby("city")["name"].first()
df.groupby("city")["age"].idxmax()      # 组内最大值所在索引
```

---

## 透视表与交叉表

```python
pd.pivot_table(df, index="city", columns="name", values="age",
               aggfunc="mean", fill_value=0, margins=True)

pd.crosstab(df["city"], df["name"])                   # 频次交叉表
pd.crosstab(df["city"], df["name"], normalize="index") # 按行归一化
pd.crosstab(df["city"], df["name"], margins=True)

pd.pivot(df, index="city", columns="name", values="age")   # 纯重塑，不聚合（唯一值需唯一）
```

| | `pivot` | `pivot_table` |
| --- | --- | --- |
| 聚合 | ❌（重复组合会报错） | ✅ |
| 参数 | index / columns / values | 同上 + `aggfunc` / `fill_value` / `margins` |

---

## 合并与连接

```python
# merge：按列/键连接（SQL 风格）
pd.merge(left, right, on="id", how="inner")
pd.merge(left, right, left_on="uid", right_on="id", how="left")
pd.merge(left, right, on=["id", "date"], how="outer")
pd.merge(left, right, on="id", suffixes=("_x", "_y"))
pd.merge(left, right, on="id", indicator=True)         # 标记行来源
pd.merge(left, right, on="id", validate="one_to_many") # 校验基数关系
pd.merge(left, right, how="cross")                     # 笛卡尔积

# join：默认按索引连接
df.join(other, how="left")
df.join(other, on="key", rsuffix="_r")

# concat：沿轴堆叠
pd.concat([df1, df2])                       # 纵向（默认 axis=0）
pd.concat([df1, df2], ignore_index=True)    # 重建 0..n-1 索引
pd.concat([df1, df2], keys=["a", "b"])      # 生成 MultiIndex 区分来源
pd.concat([df1, df2], axis=1)               # 横向按索引对齐
pd.concat([df1, df2], join="inner")         # 只保留公共列/索引

# combine_first：用右表填补左表缺失
df1.combine_first(df2)

# 时间序列专用
pd.merge_asof(df1.sort_values("t"), df2.sort_values("t"), on="t", direction="backward")
pd.merge_ordered(df1, df2, on="t", fill_method="ffill")
```

| `how` | 结果 |
| --- | --- |
| `inner` | 两表键的交集 |
| `left` | 保留左表全部行 |
| `right` | 保留右表全部行 |
| `outer` | 两表键的并集 |
| `cross` | 笛卡尔积（不指定键） |

> ⚠️ `DataFrame.append` 已在 pandas 2.0 中移除，统一改用 `pd.concat`。

---

## 重塑与变形

```python
# 宽 → 长
df.melt(id_vars=["name"], value_vars=["a", "b"],
        var_name="变量", value_name="数值")

# 长 → 宽
df.pivot(index="name", columns="变量", values="数值")
df.pivot_table(index="name", columns="变量", values="数值", aggfunc="sum")

# 堆叠
stacked = df.stack()          # 列 → 行（结果 Series / MultiIndex）
stacked.unstack()             # 行 → 列（还原）
df.stack(future_stack=True)   # 明确保留缺失组合

# 一列多值 → 多行
df.explode("tags")            # 单元格内是列表时展开

# 独热编码
pd.get_dummies(df, columns=["city"], prefix="city", drop_first=True, dtype=int)

# 其他
df.T
df.swaplevel()
df.rename_axis(columns=None)
```

---

## 字符串处理 str

`Series.str` 访问器提供向量化字符串操作，**遇 NaN 自动返回 NaN**。

```python
s = df["name"]

s.str.lower(), s.str.upper(), s.str.title()
s.str.strip(), s.str.lstrip("0"), s.str.replace("-", "", regex=False)
s.str.split(",")                     # → Series of list
s.str.split(",", expand=True)        # → 拆成多列
s.str.cat(others=df["city"], sep="-")     # 列拼接
s.str.len()                          # 长度（注意是字符数）
s.str.contains("A", na=False)        # 包含（正则默认开启）
s.str.startswith("A"), s.str.endswith("e")
s.str.extract(r"(\d+)")              # 正则提取第一组 → DataFrame
s.str.extractall(r"(\w+)")           # 提取所有匹配 → MultiIndex
s.str.findall(r"\d")
s.str.match(r"^A")                   # 从头匹配
s.str.fullmatch(r"\d{3}")            # 完整匹配
s.str.slice(0, 3)                    # 切片
s.str.pad(10, side="left", fillchar="0")
s.str.zfill(5)                       # 左补零
s.str.repeat(2)
s.str.get(0)                         # 取列表中第 0 个元素
s.str.join("-")                      # 列表元素拼接
s.str.count("a")
s.str.find("A"), s.str.rfind("A")
s.str.index("A"), s.str.rindex("A")
s.str.translate(str.maketrans("AB", "XY"))
s.str.encode("utf-8"), s.str.decode("utf-8")
s.str.normalize("NFKD")              # Unicode 归一化
```

> ⚠️ 用 `s.str.replace(..., regex=False)` 明确关闭正则（pandas 2.0 起默认 `regex=False`，但显式书写更清晰）。`str` 方法对非字符串元素返回 NaN，先 `astype("string")` 更稳。

---

## 分类类型 category

对取值重复度高的列使用 `category` 可大幅节省内存并加速分组。

```python
df["city"] = df["city"].astype("category")
df["city"] = pd.Categorical(df["city"], categories=["BJ", "SH", "GZ"], ordered=True)

df["city"].cat.categories            # 类别列表
df["city"].cat.codes                 # 整数编码
df["city"].cat.ordered               # 是否有序
df["city"].cat.add_categories(["SZ"])
df["city"].cat.remove_unused_categories()
df["city"].cat.rename_categories({"BJ": "北京"})
df["city"].cat.reorder_categories(["GZ", "SH", "BJ"])
df["city"].cat.set_categories(["BJ", "SH"], ordered=True)   # 未列出的 → NaN

# 有序类别支持比较
df["level"] = pd.Categorical(["low", "high", "mid"], categories=["low", "mid", "high"], ordered=True)
df["level"] > "low"

df["city"].value_counts()            # 默认只统计已出现的类别（pandas 2.x）
df["city"].value_counts(observed=False)   # 统计全部类别
```

> ⚠️ 分类列做 `groupby` 时默认 `observed=True`（2.x 起），不会为未出现的类别生成空组。

---

## 时间序列

**转换与构造**

```python
pd.to_datetime(df["date"])
pd.to_datetime(df["date"], format="%Y-%m-%d", errors="coerce")
pd.to_datetime(df["ts"], unit="s")                 # Unix 时间戳（秒）
pd.to_datetime("2026-01-01")                       # Timestamp
pd.Timestamp("2026-01-01 12:00")
pd.Timedelta(days=3, hours=2)
pd.Timestamp("2026-01-01") + pd.Timedelta("30D")
pd.Timestamp("2026-01-01") + pd.DateOffset(months=1)
pd.offsets.BDay(1)                                 # 下一个工作日

df["date"] = pd.to_datetime(df["date"])
df = df.set_index("date")                          # 变成 DatetimeIndex，才能用 resample
```

**频率字符串（`freq`）**

| 简写 | 含义 | 简写 | 含义 |
| --- | --- | --- | --- |
| `D` | 日 | `W` | 周（默认周天结束） |
| `B` | 工作日 | `MS` / `ME` | 月初 / 月末 |
| `h` | 小时 | `QS` / `QE` | 季初 / 季末 |
| `min` | 分钟 | `YS` / `YE` | 年初 / 年末 |
| `s` | 秒 | `15min` | 每 15 分钟 |

> ⚠️ `M`、`Q`、`Y` 等旧别名已废弃，改用 `ME`、`QE`、`YE`（`S` 结尾表示 period 起始）。

**`.dt` 访问器**

```python
df["date"].dt.year, df["date"].dt.month, df["date"].dt.day
df["date"].dt.hour, df["date"].dt.minute
df["date"].dt.dayofweek        # 0=周一
df["date"].dt.day_name()       # 'Monday'
df["date"].dt.quarter
df["date"].dt.is_month_end, df["date"].dt.is_leap_year
df["date"].dt.days_in_month
df["date"].dt.date             # → datetime.date
df["date"].dt.strftime("%Y-%m")
df["date"].dt.floor("h"), df["date"].dt.ceil("D"), df["date"].dt.round("15min")
df["date"].dt.normalize()      # 归零点，等价于 floor("D")
df["date"].dt.to_period("M")   # 转 Period
```

**重采样**

```python
df.resample("ME").sum()
df.resample("W").agg({"value": ["sum", "mean"]})
df.resample("D").ohlc()
df.resample("ME", closed="left", label="left", origin="start").sum()
df.resample("QE", kind="period").sum()      # 结果索引为 PeriodIndex
df.resample("D").asfreq()                   # 只对齐频率，不做聚合（缺失补 NaN）
df.resample("D").ffill()                    # 上采样并前向填充
```

**位移与差分**

```python
df["value"].shift(1)                    # 下移 1 行（做 lag）
df["value"].shift(-1)                   # 上移（做 lead）
df["value"].diff()                      # 一阶差分
df["value"].diff(periods=12)            # 12 期差分
df["value"].pct_change(fill_method=None)   # 变化率
df["value"].shift(freq="D")             # 按时间位移（需 DatetimeIndex）
```

**时区**

```python
df["ts"].dt.tz_localize("UTC")                     # 赋予时区（本地化）
df["ts"].dt.tz_convert("Asia/Shanghai")            # 时区转换
df.index.tz_localize("UTC").tz_convert("Asia/Shanghai")
df.tz_localize("UTC")
```

---

## 窗口计算

```python
# 滚动窗口
df["value"].rolling(window=7).mean()                     # 前 7 行均值
df["value"].rolling(window=7, min_periods=1).mean()      # 允许不足窗口也计算
df["value"].rolling(window=7, center=True).mean()
df["value"].rolling(window="30D").mean()                 # 时间窗口（需 DatetimeIndex）
df["value"].rolling(7).agg(["mean", "std", "min", "max"])
df["value"].rolling(7).apply(lambda x: x.max() - x.min(), raw=True)   # raw=True 传 ndarray，更快
df.rolling(7).corr()                                     # 滚动相关系数
df.rolling(7).cov()

# 扩展窗口（从头累积）
df["value"].expanding().mean()
df["value"].expanding(min_periods=3).sum()

# 指数加权
df["value"].ewm(span=10, adjust=False).mean()             # span 型
df["value"].ewm(alpha=0.3, adjust=False).mean()           # alpha 型
df["value"].ewm(halflife=5, adjust=False).mean()
df.ewm(span=10).std()

# 结果常接到绘图或后续列
df["ma7"] = df["value"].rolling(7).mean()
```

> ⚠️ `pct_change` 的 `fill_method` 默认值已废弃，显式传 `fill_method=None` 可避免警告与未来行为变更。

---

## 性能技巧

**1. 向量化优先**

```python
# 慢
df["c"] = [a + b for a, b in zip(df["a"], df["b"])]
# 快
df["c"] = df["a"] + df["b"]

np.where(df["a"] > 0, 1, -1)        # 条件选择
df["a"].clip(0, 100)
```

**2. 用 Arrow 后端与合适 dtype**

```python
df = pd.read_csv("data.csv", dtype_backend="pyarrow")   # 更快、更省内存
df["city"] = df["city"].astype("category")              # 高重复度列
df["id"] = pd.to_numeric(df["id"], downcast="unsigned")
df = df.convert_dtypes()
df.memory_usage(deep=True).sum() / 1024**2              # MB
```

**3. 分块处理大文件**

```python
chunks = []
for chunk in pd.read_csv("huge.csv", chunksize=500_000):
    chunks.append(chunk.groupby("city")["value"].sum())
result = pd.concat(chunks).groupby(level=0).sum()
```

**4. 避免 `iterrows`**

```python
# 最慢
for _, row in df.iterrows(): ...
# 快得多
for row in df.itertuples(index=False, name=None): ...
for col_a, col_b in zip(df["a"], df["b"]): ...      # 更快
df.to_numpy()                                        # 需要纯数值时
```

**5. `query` / `eval` 加速**

```python
df.query("a > 1 and b < 5")     # 大表上比布尔索引稍快
df.eval("c = a + b", inplace=False)
```

**6. Copy-on-Write**

pandas 3.0 起默认启用 CoW：切片返回的是共享数据的逻辑视图，**任何修改都必须显式赋值**，链式赋值不再生效。

```python
pd.options.mode.copy_on_write = True      # 提前启用（2.x）

sub = df[df["a"] > 0]
sub["b"] = 0          # ✅ 只改 sub
df.loc[df["a"] > 0, "b"] = 0              # ✅ 改原表要这样写
df[df["a"] > 0]["b"] = 0                  # ❌ 无效（链式赋值）
```

**7. 其他**

```python
pd.set_option("mode.data_manager", "block")   # 按列存，列间操作稍快
df = df.copy(deep=False)                      # 浅拷贝，慎用
del df["large_col"]                           # 及时释放
```

---

## 输出数据

```python
df.to_string()                 # 纯文本
df.to_markdown()               # Markdown 表格
df.to_dict("records")          # [{...}, {...}]
df.to_dict("list")             # {'col': [..]}
df.to_numpy()                  # ndarray
df.to_records(index=False)     # 结构化数组
df.to_period(), df.to_timestamp()

df.style.format("{:.2f}").background_gradient()   # Jupyter 美化
pd.DataFrame([...]).to_latex()                    # LaTeX 表格
```

---

## 常见坑

**1. `SettingWithCopyWarning`**

对切片结果赋值时触发，说明改动可能没写回原表：

```python
sub = df[df["a"] > 0].copy()    # ✅ 显式拷贝，明确意图
sub["b"] = 1
```

**2. `loc` 与 `iloc` 的端点差异**

```python
df.loc[0:2]     # 包含标签 2
df.iloc[0:2]    # 不含位置 2
```

**3. 布尔运算不能用 `and` / `or`**

```python
df[(df.a > 0) & (df.b < 1)]     # ✅
df[(df.a > 0) and (df.b < 1)]   # ❌
```

**4. `==` 与 `is`**

```python
df["city"] == "BJ"              # ✅ 逐元素比较
df["city"] is "BJ"              # ❌
```

**5. 已移除/废弃的 API**

| 旧写法 | 新写法 |
| --- | --- |
| `df.append(other)` | `pd.concat([df, other])` |
| `df.applymap(f)` | `df.map(f)`（2.1+） |
| `df.iteritems()` | `df.items()` |
| `df.fillna(method="ffill")` | `df.ffill()` |
| `resample("M")` | `resample("ME")` |
| `pct_change()` 默认填充 | `pct_change(fill_method=None)` |
| `df.swapaxes()` | `df.transpose()` |
| `pd.Series.append()` | `pd.concat` |
| `inplace=True` | 显式赋值（社区已不推荐） |

**6. `groupby` 默认丢弃 NaN 分组**

```python
df.groupby("city", dropna=False).size()   # 保留 NaN 组
```

**7. 聚合后的列名是 MultiIndex**

```python
res = df.groupby("city").agg({"age": ["mean", "max"]})
res.columns = ["_".join(c) for c in res.columns]      # 展平
# 或直接用命名聚合避免该问题
```

**8. 索引对齐导致的意外 NaN**

```python
s1 + s2                    # 按索引对齐，不重合处为 NaN
s1.add(s2, fill_value=0)   # 需要填充时用带 fill_value 的算术方法
```

**9. 用链式比较需加括号**

```python
df[(df.age > 20) & (df.age < 40)]
```

**10. `inplace=True` 与链式调用冲突**

`inplace` 返回 `None`，不能继续链式操作，且未来版本会逐步淘汰；建议用 `df = df.xxx(...)`。

```python
df = df.sort_values("age").reset_index(drop=True)
```

**11. 中文与编码**

```python
pd.read_csv("data.csv", encoding="gbk")            # 国内常见编码
df.to_csv("out.csv", encoding="utf-8-sig")         # Excel 打开不乱码
```

**12. 浮点精度与显示**

```python
pd.set_option("display.precision", 4)
np.allclose(a, b)          # 浮点比较，而非 ==
df.round(2)
```

---

## 参考

- pandas 官方文档：https://pandas.pydata.org/docs/
- 用户指南：https://pandas.pydata.org/docs/user_guide/index.html
- API 速查：https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf
- 10 分钟入门：https://pandas.pydata.org/docs/user_guide/10min.html
- pandas 3.0 迁移指南：https://pandas.pydata.org/docs/whatsnew/index.html
- 与 pandas 无关但常配套使用：NumPy / SciPy 速查表见本仓库同级文件
