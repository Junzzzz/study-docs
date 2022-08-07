# 第4讲 课后题

原问题：近和远平面之间的任何一点在经过“挤压”变换后， Z坐标的值会如何变化？[^1]

又因为考虑到摄像机是朝向-Z方向，$n_z > f_z$，会导致比较混乱，因此提出了一个简化后的问题：

点 $(x,y,\frac{n+f}{2})$ 在经过变换后会更靠近 **近平面** 还是 **远平面** ？

## 解

虽然好久没写过数学题了，还是试着写一写看看。

### 简化后的题解

在视频中已经可以得到矩阵：

${\displaystyle{\begin{aligned}
M_{persp \rightarrow ortho}=\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0
\end{bmatrix}
\end{aligned}}}$

右乘列向量 $(x,y,\frac{n+f}{2},1)$ ，可得变换后的值：

${\displaystyle{\begin{aligned}
z' = \frac{(n+f)^2}{2}-nf = \frac{n^2+f^2}{2}
\end{aligned}}}$

显然：

${\displaystyle{\begin{aligned}
z' > 0 > z = \frac{n+f}{2} (f < n < 0)
\end{aligned}}}$

让我困惑的是变换后的Z坐标值居然大于0，有点不可思议，不知道是哪里理解错了

有看到这里，有答案的同学请在ISSUE里发一下，感谢

[^1]: [BILI - 第4讲视频](https://www.bilibili.com/video/BV1X7411F744?p=4)