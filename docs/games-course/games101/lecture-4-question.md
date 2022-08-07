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

右乘列向量 $(x,y,\frac{n+f}{2},1)$ ，可得变换后的列向量：

${\displaystyle{\begin{aligned}
(nx,ny,\frac{n^2+f^2}{2},\frac{n+f}{2})
\end{aligned}}}$

即：

${\displaystyle{\begin{aligned}
z' = \frac{\frac{n^2+f^2}{2}}{\frac{n+f}{2}} = \frac{n^2+f^2}{n+f}
\end{aligned}}}$


两坐标相减可得：

${\displaystyle{\begin{aligned}
z' - z = \frac{n^2+f^2}{n+f} - \frac{n+f}{2} = \frac{(n-f)^2}{2(n+f)}
\end{aligned}}}$

由于摄像机朝向Z轴 **负方向** ，所以 $f < n < 0$，得

${\displaystyle{\begin{aligned}
z' - z < 0
\end{aligned}}}$

所以更靠近 **远平面**

### 原题解

上面代入的是中点这个特殊值，下面试着求一下通用解。

设两平面中间任意一点坐标为 $(x,y,z)$，那么$f < z < n < 0$

同上述解法可得变换后的坐标 $z'$ 为：

${\displaystyle{\begin{aligned}
z' = \frac{(n+f)z - nf}{z}
\end{aligned}}}$

与原坐标相减得：

${\displaystyle{\begin{aligned}
z' - z = \frac{(n+f)z - nf}{z} - z = (n+f) - (\frac{nf}{z} + z)
\end{aligned}}}$

令 $t = nf$，则：

${\displaystyle{\begin{aligned}
z' - z = (n + \frac{t}{n}) - (z + \frac{t}{z})
\end{aligned}}}$

令 ${\displaystyle{\begin{aligned}f(x)=x + \frac{t}{x}\end{aligned}}}$，则 $x \in (-\infty,-\sqrt{t})$ 时，单调递增，$x \in (-\sqrt{t},0)$ 时，单调递减。

令
${\displaystyle{\begin{aligned}
f = n - k (k > 0),
g(n)=n - \left(-\sqrt{n(n-k)}\right)
\end{aligned}}}$，对 $g(n)$ 求导得：

${\displaystyle{\begin{aligned}
g'(n) = \frac{n - \frac{1}{2}k}{\sqrt{n^2-kn}} + 1 < 0 
\end{aligned}}}$ （可以接着求导来证）

因此 $g(n)$ 在 $n < 0$ 时单调递减，且 $g(n) > 0$

则：$n > -\sqrt{n(n-k)} = -\sqrt{t}$

同理得：$f = n - k > -\sqrt{n(n-k)} = -\sqrt{t}$

所以：
${\displaystyle{\begin{aligned}
z' - z = f(n) - f(z) < 0
\end{aligned}}}$

所以更靠近 **远平面**

!!! warning "警告"

    虽然试着证明了一下，但不保证证明过程正确性，不过大致应该没问题，如果看到这里的同学发现有误，可以发下 `Issue`。

[^1]: [BILI - 第4讲视频](https://www.bilibili.com/video/BV1X7411F744?p=4)