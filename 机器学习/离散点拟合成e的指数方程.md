## 将坐标上的点拟合成e的指数方程
```python
import matplotlib.pyplot as plt
import numpy as np
from scipy.optimize import curve_fit
from matplotlib.ticker import AutoMinorLocator, MultipleLocator, FuncFormatter


# 生成e指数函数
def func(x, a, b):
    return a*np.exp(b*x) - a


# 生成x,y数据
x = np.linspace(0.6, 1.7, 23)
y = func(x, 0.5, 2.0)
y = y + 0.1 * np.random.randn(len(x))
# 拟合
popt, pcov = curve_fit(func, x, y)
# 获得拟合后的参数
a, b = popt[0], popt[1]
# 计算拟合所得数据
y2 = func(x, a, b)
# 获取当前子图
ax = plt.gca()
# set x,y limits
ax.set_xlim(0.5, 1.8)
ax.set_ylim(0, 16)
# set x,y major_tick_locator
ax.xaxis.set_major_locator(MultipleLocator(0.2))
ax.yaxis.set_major_locator(MultipleLocator(4))
# set x,y minor_tick_locator
ax.xaxis.set_minor_locator(AutoMinorLocator(2))
ax.yaxis.set_minor_locator(AutoMinorLocator(2))
# set tick format
ax.tick_params(which="major", length=8, width=2, colors="black", labelsize=18, pad=10)
ax.tick_params(which="minor", length=4, width=1.2, colors="black", labelsize=18)

# set width of axis
ax.spines['bottom'].set_linewidth(2)
ax.spines['left'].set_linewidth(2)
ax.spines['right'].set_linewidth(2)
ax.spines['top'].set_linewidth(2)

plt.text(0.05, 0.8, r"$y$ = $%1.3f*e^{%1.3f*x} - %1.3f$" %(a, b, a), fontsize=20, transform=ax.transAxes)
plt.text(0.05, 0.7, r"$a$ = %1.3f" %a, fontsize=20, transform=ax.transAxes)
plt.text(0.05, 0.6, r"$b$ = %1.3f" %b, fontsize=20, transform=ax.transAxes)

plot1 = plt.scatter(x, y, color="green", label='original values', zorder=2, marker="o", linewidth=5)

plot2 = plt.plot(x, y2, color="red", ls="-", label='curve_fit values', zorder=1, linewidth=2)

plt.xlabel(r'$x$', fontsize=20)
plt.ylabel(r'$y$', fontsize=20)
ax.legend(loc=(0.48, 0.05), ncol=1, frameon = False, scatterpoints=1, numpoints=1, prop={'size': 20})

plt.savefig('Exp-Fit.png', bbox_inches='tight', dpi=100)
plt.show()
```