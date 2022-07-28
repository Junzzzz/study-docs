# 物体旋转

在游戏中对物体进行旋转时，只需要将旋转矩阵与其所有的顶点向量相乘，就可以得到所有变换后的顶点坐标了

## 旋转矩阵

一行公式即可理解：$R\cdot\vec{v}=\vec{v}'$

其中，$R$ 为旋转矩阵，$\vec{v}$ 为初始向量，$\vec{v}'$ 为变换后向量

### 二维旋转矩阵

以逆时针旋转角度 $\theta$ 作为正角，旋转矩阵为：

$M(\theta) =  \begin{bmatrix}  \cos \theta & -\sin \theta \\  \sin \theta & \cos \theta \end{bmatrix}$

### 三维旋转矩阵

以右手笛卡尔坐标系为例，$R_x,R_y,R_z$ 分别为以 $x,y,z$ 轴旋转的旋转矩阵

$R_x(\theta_x) =  \begin{bmatrix}  1 & 0 & 0 \\  0 & \cos \theta_x & -\sin \theta_x \\  0 & \sin \theta_x & \cos \theta_x \end{bmatrix}$

$R_y(\theta_y) =  \begin{bmatrix} \cos \theta_y & 0 & \sin \theta_y \\ 0 & 1 & 0 \\ -\sin \theta_y & 0 & \cos \theta_y \end{bmatrix}$

$R_z(\theta_z) =  \begin{bmatrix} \cos \theta_z & -\sin \theta_z & 0 \\ \sin \theta_z & \cos \theta_z & 0 \\ 0 & 0 & 1 \end{bmatrix}$

## 欧拉角(Tait-Bryan角)[^1]

一种常用的旋转顺序为 $z-y-x$ 或者说 $z-y'-x''$ (表示新的轴)，绕这三个轴旋转的角依次被称作偏航角(yaw)、俯仰角(pitch)、滚动角(roll)记作 $\alpha,\beta,\gamma$

则其旋转矩阵为：

${\displaystyle{\begin{aligned}
R=R_{z}(\alpha)\,R_{y}(\beta)\,R_{x}(\gamma)&={\overset {\text{yaw}}{\begin{bmatrix}\cos \alpha &-\sin \alpha &0\\\sin \alpha &\cos \alpha &0\\0&0&1\\\end{bmatrix}}}{\overset {\text{pitch}}{\begin{bmatrix}\cos \beta &0&\sin \beta \\0&1&0\\-\sin \beta &0&\cos \beta \\\end{bmatrix}}}{\overset {\text{roll}}{\begin{bmatrix}1&0&0\\0&\cos \gamma &-\sin \gamma \\0&\sin \gamma &\cos \gamma \\\end{bmatrix}}}\\&={\begin{bmatrix}\cos \alpha \cos \beta &\cos \alpha \sin \beta \sin \gamma -\sin \alpha \cos \gamma &\cos \alpha \sin \beta \cos \gamma +\sin \alpha \sin \gamma \\\sin \alpha \cos \beta &\sin \alpha \sin \beta \sin \gamma +\cos \alpha \cos \gamma &\sin \alpha \sin \beta \cos \gamma -\cos \alpha \sin \gamma \\-\sin \beta &\cos \beta \sin \gamma &\cos \beta \cos \gamma \\
\end{bmatrix}}
\end{aligned}}}$

但是使用欧拉角进行变换会出现万向锁的情况。

!!! tip "万向锁"

    由于是根据变换后的轴进行旋转的，其中一条轴在旋转 $90°$ 后，其余两条轴会重叠，这样其中一条轴就会失去作用。

## 轴-角对

顾名思义，以任意向量为轴，将物体进行旋转，但是坐标变换的计算还是需要通过矩阵或者四元数的方式进行

### 四元数[^2] [^3]

若任意向量 $\vec{v}$ 沿着以 **单位向量** 定义的旋转轴 $\vec{u}$ 旋转 $\theta$ 角度之后的 $\vec{v}'$ 

令 $a = \cos \frac {\theta}{2}$，$b = \sin \frac{\theta}{2} \cdot \vec{u_x}$，$c = \sin \frac{\theta}{2} \cdot \vec{u_y}$，$d = \sin \frac{\theta}{2} \cdot \vec{u_z}$，$v=[0, \vec{v}]$，$v'=[0,\vec{v}']$

则四元数 $q = a + bi + cj + dk$ （ $a$ 为实部，$i,j,k$ 为虚部）

${\displaystyle{\begin{aligned}
v' = qvq^* &= \begin{bmatrix}
a & -b & -c & -d \\
b & a & -d & c \\
c & d & a & -b \\
d & -c & b & a
\end{bmatrix}
\begin{bmatrix}
a & b & c & d \\
-b & a & -d & c \\
-c & d & a & -b \\
-d & -c & b & a
\end{bmatrix}
v\\
&=\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 - 2c^2 - 2d^2 & 2bc - 2ad & 2ac + 2bd \\ 
0 & 2bc + 2ad & 1 - 2b^2 - 2d^2 & 2cd - 2ab \\
0 & 2bd - 2ac & 2ab + 2cd & 1 - 2b^2 - 2c^2
\end{bmatrix}v
\end{aligned}}}$

化简得：

${\displaystyle{\begin{aligned}
\vec{v}' = \begin{bmatrix}
1 - 2c^2 - 2d^2 & 2bc - 2ad & 2ac + 2bd \\ 
2bc + 2ad & 1 - 2b^2 - 2d^2 & 2cd - 2ab \\
2bd - 2ac & 2ab + 2cd & 1 - 2b^2 - 2c^2
\end{bmatrix} \vec{v}
\end{aligned}}}$

这样就得到了 **轴-角对** 的旋转矩阵

不过在实际应用中，直接使用四元数与三维向量转换成的纯四元数相乘即可，使用旋转矩阵运算效率并不高

同时，对于四元数特性，可以比较容易的对四元数进行插值，且不会出现万向锁的问题

!!! summary "总结"

    1. 简单的来说，上述中的四元数 $q$ 可以理解成一个变换函数，作用就是对向量进行旋转。
    2. 四元数不能简单的看作四维向量，像交换律之类的只能在特定条件才满足。
    3. 感觉不理解问题也不大，开发游戏过程中会调API就行。

[^1]:
    [Wiki-旋转矩阵](https://en.wikipedia.org/wiki/Rotation_matrix)
[^2]:
    [Wiki-四元数](https://en.wikipedia.org/wiki/Quaternion)
[^3]:
    [krasjet-四元数与三维旋转](https://krasjet.github.io/quaternion/quaternion.pdf)（强烈推荐）