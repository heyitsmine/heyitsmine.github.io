---
title: matplotlib绘制热力图动画
date: 2020-10-07 15:01:35
categories:
- python
tags:
- python
- matplotlib
---

使用python的matplotlib库绘制热力图动画

<!--more-->

# 绘制热力图

使用`matplotlib.pyplot.imshow()`方法，参考https://www.zhihu.com/question/47456526/answer/317273113

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.random.rand(101, 101)
fig, ax = plt.subplots()
img = ax.imshow(x, cmap=plt.cm.hot, origin='lower')
plt.colorbar()
plt.show()
```

# 绘制动画

使用`matplotlib.animation import FuncAnimation`, 基本结构如下：

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from mpl_toolkits.axes_grid1 import make_axes_locatable

fig, ax = plt.subplots()
div = make_axes_locatable(ax)
cax = div.append_axes('right', '5%', '5%')

num_frames = 50
row, col = 101, 101
data = np.random.rand(num_frames, row, col)
img = ax.imshow(np.zeros((row, col)), cmap=plt.cm.hot, vmin=0, vmax=1.0, origin='lower')
cbr = fig.colorbar(img, cax=cax)

def update(i):
    X = data[i]
    vmax = np.max(X)
    vmin = np.min(X)
    img.set_data(X)
    img.set_clim(vmin, vmax)

ani = FuncAnimation(fig, update, frames=num_frames)
plt.show()

```

`FuncAnimation`的参数`update`绘制每一帧的图像，参数`frames`用于产生每次调用`update`的参数。

