# SciPy 基础速查表

> 约定：`import numpy as np`，`import scipy as sp`
>
> 🏠 [返回笔记索引](README.md) ｜ 相关：[NumPy 速查表](numpy_basics.md) · [pandas 速查表](pandas_basics.md)

SciPy 是建立在 NumPy 之上的科学计算库，按子模块组织功能。绝大多数模块以**接收并返回 NumPy 数组**的方式工作。

## 目录

- [安装与子模块](#安装与子模块)
- [常量与特殊函数](#常量与特殊函数)
- [优化 optimize](#优化-optimize)
- [插值 interpolate](#插值-interpolate)
- [积分与微分方程 integrate](#积分与微分方程-integrate)
- [线性代数 linalg](#线性代数-linalg)
- [稀疏矩阵 sparse](#稀疏矩阵-sparse)
- [信号处理 signal](#信号处理-signal)
- [统计 stats](#统计-stats)
- [空间算法 spatial](#空间算法-spatial)
- [聚类 cluster](#聚类-cluster)
- [图像处理 ndimage](#图像处理-ndimage)
- [傅里叶变换 fft](#傅里叶变换-fft)
- [文件 IO io](#文件-io-io)
- [常见坑与技巧](#常见坑与技巧)

---

## 安装与子模块

```bash
pip install scipy
```

```python
import scipy as sp
sp.__version__

# 推荐按需导入子模块（比 from scipy import * 清晰）
from scipy import (
    constants, special,      # 常量、特殊函数
    optimize, interpolate,   # 优化、插值
    integrate,               # 积分、ODE
    linalg, sparse,          # 线代、稀疏矩阵
    signal,                  # 信号处理
    stats, spatial, cluster, # 统计、空间、聚类
    ndimage, fft, io,        # 图像、FFT、IO
)
```

| 子模块 | 用途 |
| --- | --- |
| `constants` | 物理常量、单位换算 |
| `special` | 特殊数学函数（Γ、Bessel、误差函数等） |
| `optimize` | 最小化、求根、曲线拟合、线性规划 |
| `interpolate` | 一维/多维插值、样条 |
| `integrate` | 数值积分、常微分方程求解 |
| `linalg` | 线性代数（比 `numpy.linalg` 更全、更快） |
| `sparse` | 稀疏矩阵及其线性代数 |
| `signal` | 滤波、卷积、谱分析、峰值检测 |
| `stats` | 概率分布、统计检验、描述统计 |
| `spatial` | 距离、KD 树、凸包、Delaunay、旋转 |
| `cluster` | 层次聚类、k-means、矢量量化 |
| `ndimage` | N 维图像滤波、形态学、标记 |
| `fft` | 比 `numpy.fft` 更快，支持多线程与实数优化 |
| `io` | MATLAB、WAV、Matrix Market、NetCDF |

---

## 常量与特殊函数

**`scipy.constants`**

```python
from scipy.constants import pi, c, h, k, G, g, e, golden

c                      # 299792458.0   光速 (m/s)
g                      # 9.80665       标准重力加速度
k                      # 1.380649e-23  玻尔兹曼常数

constants.physical_constants["proton mass"]   # (值, 单位, 不确定度)
constants.unit("eV", "J")                     # 单位换算因子
constants.convert_temperature(100, "C", "K")  # 373.15
constants.gram, constants.kilo, constants.micro   # 前缀因子
```

**`scipy.special`**

```python
x = np.array([0.5, 1.0, 2.0])

special.gamma(x)          # Γ 函数（阶乘的推广）
special.gammaln(x)        # ln|Γ(x)|（大值更稳定）
special.beta(2, 3)        # B 函数
special.factorial(5)      # 120.0
special.comb(5, 2)        # 10.0   组合数
special.perm(5, 2)        # 20.0   排列数

special.erf(x)            # 误差函数
special.erfc(x)           # 互补误差函数 1 - erf(x)
special.erfinv(0.5)       # 反误差函数

special.expit(x)          # sigmoid: 1/(1+e^-x)
special.logit(x)          # logit: ln(x/(1-x))
special.softmax(x)        # softmax
special.logsumexp(x)      # ln(Σe^x)，防溢出

special.jv(1, x)          # 第一类 Bessel 函数
special.yn(1, x)          # 第二类 Bessel 函数
special.eval_legendre(2, x)   # Legendre 多项式
special.lambertw(1)       # Lambert W 函数
```

---

## 优化 optimize

**无约束 / 有约束最小化**

```python
from scipy.optimize import minimize, minimize_scalar

def f(x):
    return (x[0] - 1) ** 2 + (x[1] + 2) ** 2

res = minimize(f, x0=[0, 0], method="BFGS")
res.x        # array([ 1., -2.])   最优解
res.fun      # 6.66e-16           最优值
res.success  # True
res.message  # 收敛信息
res.nit      # 迭代次数

# 常用 method
# "Nelder-Mead"  无导数，鲁棒但慢
# "BFGS"         拟牛顿，需梯度（可自动数值求导）
# "L-BFGS-B"     支持 bounds（变量上下界）
# "SLSQP"        支持等式/不等式约束，含 bounds
# "trust-constr" 通用约束优化，最稳但配置复杂

# 变量上下界
minimize(f, [0, 0], bounds=[(-1, 1), (-3, 0)])

# 不等式约束 c(x) >= 0
cons = {"type": "ineq", "fun": lambda x: 1 - x[0] - x[1]}
minimize(f, [0, 0], constraints=cons)

# 一维搜索
minimize_scalar(lambda t: (t - 3) ** 2, bounds=(0, 5), method="bounded")
```

**求根**

```python
from scipy.optimize import root, fsolve, brentq, newton

root(lambda x: x ** 3 - 1, x0=1).x        # 多维/标量统一接口
fsolve(lambda x: x ** 2 - 4, x0=3)        # 旧式接口（仍常用）
brentq(lambda x: x ** 2 - 4, 0, 5)        # 区间内有符号变化时最稳
newton(lambda x: x ** 2 - 4, x0=3)        # 牛顿法，可传 fprime
```

**曲线拟合**

```python
from scipy.optimize import curve_fit, least_squares

def model(x, a, b):
    return a * np.exp(-b * x)

xdata = np.linspace(0, 4, 30)
ydata = model(xdata, 2.5, 1.3) + 0.1 * np.random.default_rng(0).normal(size=30)

popt, pcov = curve_fit(model, xdata, ydata, p0=[1, 1])
popt                  # 拟合参数 [a, b]
perr = np.sqrt(np.diag(pcov))   # 参数标准差

# 带权拟合
curve_fit(model, xdata, ydata, sigma=np.ones_like(ydata) * 0.1, absolute_sigma=True)

# 通用最小二乘（可加 bounds / loss 鲁棒核）
least_squares(lambda p: model(xdata, *p) - ydata, x0=[1, 1],
              bounds=([0, 0], [10, 10]), loss="soft_l1")
```

**全局优化**

```python
from scipy.optimize import differential_evolution, dual_annealing, shgo

bounds = [(-5, 5), (-5, 5)]
res = differential_evolution(f, bounds, seed=42)
dual_annealing(f, bounds)
shgo(f, bounds)          # 单纯形同调，适合低维全局最优
```

**线性规划 / 整数规划**

```python
from scipy.optimize import linprog, milp, LinearConstraint, Bounds

# min c @ x  s.t.  A_ub @ x <= b_ub,  A_eq @ x == b_eq
c = [-1, -2]                     # 最大化 x0 + 2*x1，取负号后最小化
A_ub = [[1, 1], [1, -1]]
b_ub = [4, 1]
res = linprog(c, A_ub=A_ub, b_ub=b_ub, bounds=[(0, None), (0, None)])

# 整数规划（SciPy >= 1.9）
from scipy.optimize import milp
milp(c=c, constraints=LinearConstraint(A_ub, -np.inf, b_ub),
     integrality=np.ones(2), bounds=Bounds(0, np.inf))
```

**其他**

```python
from scipy.optimize import nnls, brentq, approx_fprime

nnls(A, b)                          # 非负最小二乘
approx_fprime(x0, f, 1e-8)          # 数值梯度
```

---

## 插值 interpolate

```python
from scipy.interpolate import (
    CubicSpline, make_interp_spline, PchipInterpolator,
    RegularGridInterpolator, griddata, RBFInterpolator,
)
import numpy as np

x = np.linspace(0, 10, 11)
y = np.sin(x)

# 一维：优先用这些（interp1d 已属遗留 API）
np.interp(3.5, x, y)                        # 最快，仅线性
cs = CubicSpline(x, y)
cs(3.5)                                     # 三次样条求值
cs(x_new, 1)                                # 求一阶导数
cs.integrate(0, 10)                         # 积分
cs.roots()                                  # 零点

spl = make_interp_spline(x, y, k=3)         # B 样条（可调阶数 k）
PchipInterpolator(x, y)(3.5)                # 保形分段三次，无过冲

# 二维规则网格
xi = np.linspace(0, 10, 50)
yi = np.linspace(0, 10, 50)
X, Y = np.meshgrid(xi, yi)
Z = np.sin(X) * np.cos(Y)

rgi = RegularGridInterpolator((xi, yi), Z, method="linear")   # 也支持 "cubic" / "nearest"
rgi(np.array([[2.5, 3.5]]))

# 散点数据
from scipy.interpolate import LinearNDInterpolator, NearestNDInterpolator
pts = np.random.default_rng(0).random((100, 2)) * 10
vals = np.sin(pts[:, 0]) + np.cos(pts[:, 1])

LinearNDInterpolator(pts, vals)([[2.5, 3.5]])
griddata(pts, vals, (2.5, 3.5), method="cubic")   # 也支持 "linear" / "nearest"
RBFInterpolator(pts, vals, kernel="thin_plate_spline")([[2.5, 3.5]])

# 平滑样条（带平滑因子 s）
from scipy.interpolate import UnivariateSpline
UnivariateSpline(x, y + 0.05 * np.random.default_rng(0).normal(size=11), s=0.1)(3.5)
```

> ⚠️ `interp1d` 是遗留接口，新代码建议用 `np.interp`（线性）、`CubicSpline` 或 `make_interp_spline`。

---

## 积分与微分方程 integrate

**数值积分**

```python
from scipy.integrate import quad, dblquad, tplquad, nquad, trapezoid, simpson

# 定积分 ∫ f dx —— 返回 (积分值, 绝对误差估计)
val, err = quad(lambda x: np.exp(-x ** 2), 0, np.inf)   # → (0.8862, ...)

# 带参数：函数第一个参数为自变量，其余通过 args 传入
quad(lambda x, a: np.sin(a * x), 0, np.pi, args=(2,))

# 指定不连续点/奇异点，显著提升精度
quad(lambda x: 1 / np.sqrt(abs(x)), -1, 1, points=[0])

# 多重积分
dblquad(lambda y, x: x * y, 0, 1, 0, 2)                  # 注意 y 在前
tplquad(lambda z, y, x: x * y * z, 0, 1, 0, 1, 0, 1)
nquad(lambda x, y: x * y, [[0, 1], [0, 1]])               # 一般 n 维

# 采样数据积分
x = np.linspace(0, np.pi, 101)
y = np.sin(x)
trapezoid(y, x)              # 梯形法（np.trapz 已在 NumPy 2.0 移除）
simpson(y, x=x)              # 辛普森法，精度更高
```

**常微分方程**

```python
from scipy.integrate import solve_ivp, odeint

# dy/dt = -k*y  →  解析解 y = y0 * exp(-k t)
def dydt(t, y, k):
    return -k * y

sol = solve_ivp(dydt, t_span=(0, 5), y0=[10.0], args=(1.5,),
                method="RK45",        # 非刚性: "RK45"/"DOP853"
                                      # 刚性:   "Radau"/"BDF"/"LSODA"
                t_eval=np.linspace(0, 5, 100),
                rtol=1e-6, atol=1e-9)

sol.t        # 时间点
sol.y        # 形状 (n_state, n_time)
sol.success

# 事件检测（过零点、撞击时间等）
def hit(t, y, k):
    return y[0] - 1.0
hit.terminal = True          # 触发后停止

solve_ivp(dydt, (0, 20), [10.0], args=(1.5,), events=hit)

# 旧式接口（仍可用）
odeint(dydt, [10.0], np.linspace(0, 5, 100), args=(1.5,))

# 两点边值问题
from scipy.integrate import solve_bvp
```

| method | 适用场景 |
| --- | --- |
| `RK45` / `DOP853` | 非刚性，默认选择 |
| `Radau` / `BDF` | 刚性方程组 |
| `LSODA` | 自动切换刚性/非刚性 |
| `odeint` | 遗留接口，`LSODA` 内核 |

---

## 线性代数 linalg

`scipy.linalg` 是 `numpy.linalg` 的超集，底层直接调用 LAPACK，通常更快且提供更多分解。

```python
from scipy import linalg

A = np.array([[3.0, 1.0], [1.0, 2.0]])
b = np.array([9.0, 8.0])

linalg.solve(A, b)                # 解 Ax = b
linalg.inv(A)                     # 逆矩阵（尽量用 solve 替代）
linalg.det(A)                     # 行列式
linalg.norm(A)                    # 范数（ord=1/2/inf/'fro'）
linalg.pinv(A)                    # 伪逆
linalg.lstsq(A, b)                # 最小二乘

# 分解
P, L, U = linalg.lu(A)            # LU 分解
Q, R = linalg.qr(A)               # QR 分解
U, S, Vt = linalg.svd(A)          # 奇异值分解
L = linalg.cholesky(A)            # Cholesky（A 需对称正定）
w, v = linalg.eig(A)              # 一般特征值
w, v = linalg.eigh(A)             # 对称/厄米矩阵（更快更稳，推荐）

# 分解一次、多次求解（避免重复分解）
lu, piv = linalg.lu_factor(A)
linalg.lu_solve((lu, piv), b)

cho = linalg.cho_factor(A)
linalg.cho_solve(cho, b)

# 矩阵函数
linalg.expm(A)                    # 矩阵指数 e^A
linalg.logm(A)                    # 矩阵对数
linalg.sqrtm(A)                   # 矩阵平方根

# 结构矩阵
linalg.toeplitz([1, 2, 3])
linalg.hankel([1, 2, 3])
linalg.block_diag(A, A)
linalg.kron(A, A)                 # Kronecker 积
linalg.circulant([1, 2, 3])
```

| | `numpy.linalg` | `scipy.linalg` |
| --- | --- | --- |
| 依赖 | 自带 LAPACK | 自带 LAPACK + BLAS |
| 覆盖范围 | 常用分解 | 更全（lu / cho_solve / expm ...） |
| 速度 | 基准 | 通常更快 |
| 建议 | 只做基础运算时用 | 需要完整功能时用 |

---

## 稀疏矩阵 sparse

```python
from scipy import sparse
from scipy.sparse import csr_matrix, csc_matrix, coo_matrix, lil_matrix, diags, eye

# 构造
data = np.array([1.0, 2.0, 3.0])
rows = np.array([0, 1, 2])
cols = np.array([0, 2, 1])
A = coo_matrix((data, (rows, cols)), shape=(3, 3)).tocsr()

diags([[1, 2, 3]], offsets=0)          # 对角矩阵
eye(1000, format="csr")                # 稀疏单位阵
sparse.random(100, 100, density=0.01, format="csr", random_state=0)
sparse.kron(eye(3), eye(3))

# 格式转换（构造用 coo/lil，计算用 csr/csc）
A.tocsr()      # 行优先，矩阵乘法/行切片快
A.tocsc()      # 列优先，列切片快
A.tocoo()      # 三元组，便于导出
A.tolil()      # 逐元素修改方便，但慢

# 属性与查看
A.shape, A.nnz, A.dtype
A.toarray()                     # 转为稠密 ndarray（注意内存！）
A.data, A.indices, A.indptr     # CSR 内部结构
A.todense()                     # 转为 np.matrix（遗留，优先 toarray）

# 运算
A @ A                           # 矩阵乘法（稀疏）
A * 2                           # 标量与所有非零元素相乘
A.multiply(B)                   # 逐元素乘积；A * B 在稀疏矩阵中是逐元素
A + B, A - B
A.sum(), A.sum(axis=0), A.max()
A.T, A.astype(np.float32)
sparse.vstack([A, A]), sparse.hstack([A, A])
A.eliminate_zeros()             # 原地移除显式零
A.setdiag([1, 1, 1])

# 稀疏线性代数
from scipy.sparse.linalg import spsolve, spilu, cg, gmres, eigsh, svds, norm, expm_multiply

spsolve(A, b)                   # 直接法解稀疏线性方程组
spilu(A.tocsc())                # 不完全 LU 预条件
cg(A, b)                        # 共轭梯度（对称正定）
gmres(A, b, maxiter=1000)       # 广义最小残差
eigsh(A, k=5, which="LM")       # 对称稀疏矩阵前 k 个特征值
svds(A, k=5)                    # 稀疏 SVD

# IO
sparse.save_npz("A.npz", A)
A = sparse.load_npz("A.npz")
sparse.io.mmwrite("A.mtx", A)
```

> ⚠️ 稀疏矩阵（`sp_matrix`）的 `*` 是**逐元素**乘法，矩阵乘法要用 `@`。新代码可用稀疏数组 API（`csr_array`、`diags_array`）。切勿对超大规模矩阵调用 `toarray()`。

---

## 信号处理 signal

```python
from scipy import signal

# 卷积与相关
signal.convolve(x, h, mode="full")        # mode: "full" / "same" / "valid"
signal.fftconvolve(x, h, mode="same")     # FFT 加速，长序列更快
signal.oaconvolve(x, h, mode="same")      # 重叠相加，最适合长信号
signal.correlate(x, x, mode="full")

# 峰值检测
peaks, props = signal.find_peaks(x, height=0.5, distance=10, prominence=0.2)
signal.peak_widths(x, peaks, rel_height=0.5)
signal.peak_prominences(x, peaks)

# 滤波器设计（返回 b, a 或 SOS，推荐 SOS 形式）
b, a = signal.butter(4, 0.2, btype="low")            # 4 阶低通，Wn 相对于 Nyquist
sos = signal.butter(4, 0.2, btype="low", output="sos")
sos = signal.cheby1(4, 1, 0.2, btype="high", output="sos")
sos = signal.ellip(4, 1, 40, [0.1, 0.4], btype="band", output="sos")
b = signal.firwin(51, 0.3, window="hamming")

w, h = signal.freqz(b, a)                 # 频率响应
signal.freqz_sos(sos)

# 滤波
y = signal.lfilter(b, a, x)               # 单向因果滤波（引入相位延迟）
y = signal.filtfilt(b, a, x)              # 零相位（前后各滤一次），需 padlen 足够
y = signal.sosfilt(sos, x)
y = signal.sosfiltfilt(sos, x)            # 推荐组合：数值稳定 + 零相位

# 平滑与去噪
signal.detrend(x)
signal.savgol_filter(x, window_length=11, polyorder=3)   # Savitzky-Golay
signal.medfilt(x, kernel_size=5)
signal.wiener(x, mysize=5)

# 谱分析
f, Pxx = signal.periodogram(x, fs=1000)
f, Pxx = signal.welch(x, fs=1000, nperseg=256)           # 功率谱密度
f, t, Sxx = signal.spectrogram(x, fs=1000, nperseg=256)
f, t, Zxx = signal.stft(x, fs=1000, nperseg=256)
signal.istft(Zxx, fs=1000)

# 重采样
signal.resample(x, num=500)              # FFT 法重采样
signal.resample_poly(x, up=3, down=2)    # 多相滤波，抗混叠更好
signal.decimate(x, q=4)                  # 抽取（先低通）

# 解析信号
signal.hilbert(x)                        # 希尔伯特变换，可提取包络
abs(signal.hilbert(x))                   # 包络

# 窗函数
signal.get_window("hann", 51)
signal.windows.hann(51)
```

---

## 统计 stats

```python
from scipy import stats

# 描述统计
stats.describe(x)                        # 样本量、均值、方差、偏度、峰度
stats.gmean(x), stats.hmean(x)           # 几何/调和平均
stats.trim_mean(x, 0.1)                  # 截尾均值
stats.zscore(x)                          # 标准分数
stats.iqr(x), stats.sem(x)               # 四分位距、均值标准误
stats.gaussian_kde(x)                    # 核密度估计
stats.rankdata(x), stats.percentileofscore(x, 5)

# 概率分布：统一接口
# rvs 抽样 / pdf 密度 / cdf 分布 / sf 生存 / ppf 分位数 / fit 拟合
dist = stats.norm(loc=0, scale=1)
dist.rvs(size=10, random_state=0)
dist.pdf(0), dist.cdf(1.96), dist.sf(1.96)
dist.ppf(0.975), dist.interval(0.95)
dist.mean(), dist.var(), dist.std(), dist.median()
stats.norm.fit(data)                     # 估计 (loc, scale)

stats.t(df=10).ppf(0.975)                # t 分布
stats.chi2.isf(0.05, df=3)               # 卡方分布（上侧分位数）
stats.binom(n=10, p=0.5).pmf(5)          # 离散分布用 pmf
stats.poisson(mu=3).rvs(5)
stats.expon(scale=2).pdf(1)
stats.lognorm(s=1, scale=np.e).median()

# 假设检验
stats.ttest_1samp(a, popmean=0)              # 单样本 t 检验
stats.ttest_ind(a, b, equal_var=False)       # 独立两样本（Welch）
stats.ttest_rel(a, b)                        # 配对样本
stats.mannwhitneyu(a, b)                     # 非参数两样本
stats.wilcoxon(a, b)                         # 非参数配对
stats.kruskal(a, b, c)                       # 非参数多组
stats.f_oneway(a, b, c)                      # 单因素方差分析
stats.tukey_hsd(a, b, c)                     # 事后多重比较
stats.levene(a, b), stats.bartlett(a, b)     # 方差齐性
stats.shapiro(x)                             # 正态性（小样本）
stats.normaltest(x)                          # 正态性（偏度+峰度）
stats.kstest(x, "norm")                      # Kolmogorov-Smirnov
stats.anderson(x)                            # Anderson-Darling

# 分类数据检验
table = np.array([[10, 20], [30, 40]])
stats.chi2_contingency(table)                # 卡方独立性
stats.fisher_exact(table)                    # Fisher 精确检验
stats.chisquare(observed, f_exp=expected)    # 拟合优度

# 相关与回归
stats.pearsonr(x, y)                         # 线性相关 + p 值
stats.spearmanr(x, y)                        # 秩相关
stats.kendalltau(x, y)
stats.linregress(x, y)                       # 简单线性回归
                                             # → slope, intercept, rvalue, pvalue, stderr

# 置信区间与重采样
stats.t(df=n - 1).interval(0.95, loc=mean, scale=sem)
stats.bootstrap((x,), np.mean, confidence_level=0.95, random_state=0)
stats.permutation_test((a, b), lambda x, y: np.mean(x) - np.mean(y))

# 多重比较校正
stats.false_discovery_control(pvals, method="bh")   # Benjamini-Hochberg
stats.false_discovery_control(pvals, method="by")   # Benjamini-Yekutieli
```

---

## 空间算法 spatial

```python
from scipy.spatial import distance, KDTree, cKDTree, ConvexHull, Delaunay, Voronoi
from scipy.spatial.transform import Rotation

# 距离
distance.euclidean(p, q)
distance.cityblock(p, q)
distance.cosine(p, q)
distance.cdist(A, B)                # 两两距离，形状 (len(A), len(B))
distance.pdist(A)                   # 压缩的上三角距离矩阵
distance.squareform(distance.pdist(A))   # 转为方阵

# KD 树（最近邻检索）
tree = cKDTree(points)              # cKDTree 与 KDTree 接口一致，更快
d, idx = tree.query(query_points, k=3)          # k 近邻
pairs = tree.query_pairs(r=0.5)                 # 半径内点对
groups = tree.query_ball_point(query_point, r=0.5)
d, idx = tree.query(query_points, k=1, workers=-1)   # 多线程

# 几何结构
hull = ConvexHull(points)
hull.vertices, hull.volume, hull.area

tri = Delaunay(points)              # Delaunay 三角剖分
tri.simplices                        # 三角形顶点索引

vor = Voronoi(points)               # Voronoi 图
vor.points, vor.regions, vor.ridge_points

# 旋转（3D）
r = Rotation.from_euler("xyz", [90, 0, 0], degrees=True)
r.as_quat(), r.as_matrix(), r.as_euler("zyx", degrees=True)
Rotation.from_rotvec([0, 0, np.pi / 2])
r.apply(vectors)

# 其他
distance.distance_matrix(p, q)      # 带 p 范数
from scipy.spatial import procrustes, geometric_slerp
procrustes(A, B)                    # 正交 Procrustes 分析
```

---

## 聚类 cluster

```python
from scipy.cluster import hierarchy
from scipy.cluster.vq import kmeans2, vq, whiten

# 层次聚类
Z = hierarchy.linkage(X, method="ward")     # method: "single"/"complete"/"average"/"ward"
hierarchy.dendrogram(Z)                     # 绘制树状图
labels = hierarchy.fcluster(Z, t=3, criterion="maxclust")   # criterion: "distance"/"maxclust"
hierarchy.cophenet(Z)                       # 共表相关系数（聚类质量）
hierarchy.fclusterdata(X, t=3, criterion="maxclust")        # 一步到位
hierarchy.leaves_list(Z)

# 矢量量化 / k-means
centroids, labels = kmeans2(X, k=3, seed=0, minit="++")
code, dist = vq(X, centroids)               # 最近质心分配
Xw = whiten(X)                              # 按标准差归一化（k-means 前常用）
```

---

## 图像处理 ndimage

```python
from scipy import ndimage

# 滤波
ndimage.gaussian_filter(img, sigma=2)
ndimage.uniform_filter(img, size=5)
ndimage.median_filter(img, size=3)
ndimage.sobel(img, axis=0)                  # 边缘检测
ndimage.laplace(img)                         # 拉普拉斯
ndimage.prewitt(img, axis=1)

# 形态学
ndimage.binary_erosion(mask, iterations=2)
ndimage.binary_dilation(mask, iterations=2)
ndimage.binary_opening(mask)                # 先腐蚀后膨胀（去小噪点）
ndimage.binary_closing(mask)                # 先膨胀后腐蚀（填小孔）
ndimage.grey_erosion(img, size=3)

# 标记与区域分析
labels, n = ndimage.label(mask)             # 连通域标记
ndimage.sum(img, labels, index=range(1, n + 1))
ndimage.center_of_mass(img, labels, range(1, n + 1))
ndimage.find_objects(labels)
ndimage.distance_transform_edt(mask)        # 欧氏距离变换

# 几何变换
ndimage.zoom(img, zoom=2, order=1)          # 缩放
ndimage.rotate(img, angle=30, reshape=False)
ndimage.shift(img, shift=(5, -3))
ndimage.map_coordinates(img, coords, order=3)                  # 任意坐标插值
ndimage.affine_transform(img, matrix, offset=offset)           # 仿射变换
```

---

## 傅里叶变换 fft

`scipy.fft` 与 `numpy.fft` 接口几乎一致，但更快，并提供实数优化与多线程。

```python
from scipy import fft

X = fft.fft(x)                        # 复数 FFT
X = fft.rfft(x)                       # 实数输入 → 只算一半，更快
x = fft.irfft(X, n=len(x))
fft.fft2(img), fft.fftn(vol)          # 多维
fft.dct(x, type=2), fft.dst(x, type=2)   # DCT / DST
fft.fftshift(X)                       # 将零频移到中心
freq = fft.fftfreq(len(x), d=1 / fs)  # 频率轴
freq = fft.rfftfreq(len(x), d=1 / fs)

fft.get_workers()                     # 查看可用线程
fft.set_workers(4)
fft.next_fast_len(1000)               # 返回 ≥1000 的最快 FFT 长度
fft.fft(x, n=fft.next_fast_len(len(x)))   # 补零到快速长度，加速
```

---

## 文件 IO io

```python
from scipy import io

# MATLAB
data = io.loadmat("data.mat")                       # → dict
data["variable"]
io.savemat("out.mat", {"a": array_a, "b": array_b})
io.whosmat("data.mat")                              # 列出变量名、形状、类型
# MATLAB v7.3 (HDF5) 请用 h5py

# WAV 音频
rate, audio = io.wavfile.read("sound.wav")
io.wavfile.write("out.wav", rate, audio.astype(np.int16))

# Matrix Market（稀疏矩阵交换格式）
io.mmread("matrix.mtx")
io.mmwrite("matrix.mtx", sparse_mat)

# NetCDF / IDL 等
io.netcdf_file("data.nc", "r")
```

---

## 常见坑与技巧

**1. `quad` 的参数顺序**

被积函数**第一个参数必须是自变量**，其余参数用 `args=(...)` 传入；`dblquad` 中内层变量的函数签名顺序为 `f(y, x)`。

```python
quad(lambda x, a, b: a * x + b, 0, 1, args=(2, 3))
```

**2. 积分出现 `IntegrationWarning`**

被积函数在区间内不光滑或含奇点，用 `points=[...]` 指出不连续点，或放宽 `epsabs` / `epsrel`：

```python
quad(f, -1, 1, points=[0], epsabs=1e-6, limit=200)
```

**3. `interp1d` 与 `odeint` 属遗留接口**

新代码用 `CubicSpline` / `make_interp_spline` / `np.interp` 与 `solve_ivp`。

**4. `trapz` 已改名**

`np.trapz` 在 NumPy 2.0 中移除，改用 `np.trapezoid`；SciPy 侧为 `scipy.integrate.trapezoid`。

**5. 稀疏矩阵的 `*` 不是矩阵乘法**

```python
A * B          # 逐元素（Hadamard）乘积
A @ B          # 矩阵乘法 ✅
A.multiply(B)  # 逐元素，等价于 A * B
```

**6. 稀疏矩阵不要随意稠密化**

`A.toarray()` 在 10^6 × 10^6、密度 0.1% 的矩阵上会产生 ~800 GB 需求。计算一律走 `spsolve` / `eigsh` / `cg` 等稀疏例程。

**7. 优先 `eigh` 而非 `eig`**

对称/厄米矩阵用 `linalg.eigh`，更快、更稳定，且返回**升序**排序的实特征值；`eig` 对一般矩阵可能返回复数值。

**8. 滤波优先 `sos` 形式**

高阶 IIR 滤波器用 `b, a` 表示会因系数精度问题导致数值不稳定，改用二阶节串联：

```python
sos = signal.butter(8, 0.2, output="sos")
y = signal.sosfiltfilt(sos, x)
```

**9. `filtfilt` 对短信号报错**

零相位滤波需要足够长的数据（默认 `padlen = 3 * max(len(a), len(b))`），可用 `padlen=0` 或改为 `sosfiltfilt`。

**10. `curve_fit` 初值很关键**

非线性拟合对 `p0` 敏感，参数尺度差异大时应先归一化，或改用 `least_squares` 配合 `bounds` 限制搜索范围。

**11. 随机种子统一**

SciPy 各函数通过 `random_state`（统计/稀疏）或 `seed`（聚类/全局优化）控制随机性；NumPy 侧推荐 `np.random.default_rng(seed)`。

**12. `stats` 分布对象的参数化**

不同分布的参数名不同（`norm` 用 `loc/scale`，`t` 用 `df`，`lognorm` 用 `s/shape/scale`）。可用 `stats.<dist>.shapes` 查看：`stats.lognorm.shapes` → `'s'`。

**13. 版本查询**

```python
sp.__version__
np.show_config()      # 查看底层 BLAS/LAPACK 后端
```

---

## 参考

- SciPy 官方文档：https://docs.scipy.org/doc/scipy/
- 优化教程：https://docs.scipy.org/doc/scipy/tutorial/optimize.html
- 插值教程：https://docs.scipy.org/doc/scipy/tutorial/interpolate.html
- 稀疏矩阵：https://docs.scipy.org/doc/scipy/reference/sparse.html
- 统计分布速查：https://docs.scipy.org/doc/scipy/reference/stats.html
- SciPy 与 NumPy 的差异：https://docs.scipy.org/doc/scipy/reference/numpy-routines.html
