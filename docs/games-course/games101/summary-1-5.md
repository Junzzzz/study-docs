# 课程 1-5 阶段性总结

这部分内容主要介绍了：

- 向量、矩阵运算（线性代数）
- MVP变换（Model-View-Projection）
- 视口变换（Viewport）

接下来做一下简单总结，线代部分就略过了。

## M-V-P 变换

在我的理解中，完成这三种变换事实上就是将物体映射到标准空间中。

这里的标准空间是以原点为中心，边长为2的立方体空间，课程中用 $[-1,1]^3$ 来表示

### 模型变换 (Model Transoformation)

这里的变换主要是对观察物体进行操作，通过缩放、旋转、平移等操作，对物体坐标进行变换。

用官方点的话讲就是 **将模型空间转换到世界空间**。简单的来讲就是将一个物体摆放在一个世界中，以该世界的坐标系来表示该物体。

#### Scale（缩放）

${\displaystyle{\begin{aligned}
S(s_x,s_y,s_z)=\begin{bmatrix}
s_x & 0 & 0 & 0 \\
0 & s_y & 0 & 0 \\
0 & 0 & s_z & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

#### Translation（平移）

${\displaystyle{\begin{aligned}
T(t_x,t_y,t_z)=\begin{bmatrix}
1 & 0 & 0 & t_x \\
0 & 1 & 0 & t_y \\
0 & 0 & 1 & t_z \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

#### Rotation（旋转）

${\displaystyle{\begin{aligned}
R_x(\alpha)=\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & \cos\alpha & -\sin\alpha & 0 \\
0 & \sin\alpha & \cos\alpha & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

${\displaystyle{\begin{aligned}
R_y(\alpha)=\begin{bmatrix}
\cos\alpha & 0 & \sin\alpha & 0 \\
0 & 1 & 0 & 0 \\
-\sin\alpha & 0 & \cos\alpha & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

${\displaystyle{\begin{aligned}
R_z(\alpha)=\begin{bmatrix}
\cos\alpha & -\sin\alpha & 0 & 0 \\
\sin\alpha & \cos\alpha & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

### 观察变换 (View / Camera Transoformation)

观察变换就是 **将世界空间转换到观察空间**。

由于摄像机（观察者）与物体相对位置是固定的，因此将摄像机摆放在原点位置，朝-Z的方向观看，同时物体位置跟着变换，这样会有很多好处（没有具体说明，大概是方便计算之类的）。

#### 变换公式

 - 位置：$\vec{e}$
 - 观察方向：$\hat{g}$
 - 向上方向：$\hat{t}$

${\displaystyle{\begin{aligned}
M_{view}&=R_{view}T_{view} \\
&=\begin{bmatrix}
x_{\hat{g}\times\hat{t}} & y_{\hat{g}\times\hat{t}} & z_{\hat{g}\times\hat{t}} & 0 \\
x_t & y_t & z_t & 0 \\
x_{-g} & y_{-g} & z_{-g} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -x_e \\
0 & 1 & 0 & -y_e \\
0 & 0 & 1 & -z_e \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

其实相当于是对坐标系进行变换

### 投影变换 (Projection Transoformation)

投影变换就是 **将观察空间转换到裁剪空间**。

由于人观察物体的视野范围是一个锥形，映射物体到标准立方体空间时会产生一定的扭曲，反应在屏幕上就是有“近大远小”，平行线交于一点的透视效果。

#### 变换公式

${\displaystyle{\begin{aligned}
M_{ortho}=\begin{bmatrix}
\frac{2}{r-l} & 0 & 0 & 0 \\
0 & \frac{2}{t-b} & 0 & 0 \\
0 & 0 & \frac{2}{n-f} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -\frac{r+l}{2} \\
0 & 1 & 0 & -\frac{t+b}{2} \\
0 & 0 & 1 & -\frac{n+f}{2} \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$

${\displaystyle{\begin{aligned}
M_{persp \rightarrow ortho}=\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0
\end{bmatrix}
\end{aligned}}}$

${\displaystyle{\begin{aligned}
M_{persp}=M_{ortho}M_{persp \rightarrow ortho}
\end{aligned}}}$

对于 $M_{persp \rightarrow ortho}$ 矩阵，可能会发现与一些教科书上的公式不太一样，主要是这里视频中规定的 $f<n<0$，而其他教材中 $f>n>0$，会出现正负号不太一样的地方

#### 常用表示

一般定义视锥会用到两个参数 $fovY$（垂直可视角度）、 $aspect$（宽高比）

${\displaystyle{\begin{aligned}
\tan\frac{fovY}{2}=\frac{t}{\lvert n \rvert}
\end{aligned}}}$

${\displaystyle{\begin{aligned}
aspect=\frac{r}{t}
\end{aligned}}}$

## 视口变换

忽略先前的标准立方体空间中的 $Z$ 轴，只对 $x,y$ 平面内进行变换，将 $[-1,1]^2$ 转换到 $[0,width]\times[0,height]$ 

${\displaystyle{\begin{aligned}
M_{viewport}=\begin{bmatrix}
\frac{width}{2} & 0 & 0 & \frac{width}{2} \\
0 & \frac{height}{2} & 0 & \frac{height}{2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{aligned}}}$
