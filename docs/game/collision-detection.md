# 碰撞检测

## SAT分离轴理论

**SAT** (分离轴理论) 是 **凸集分离定理** 在碰撞检测中的一种运用。

理论描述[^1]如下：

如果存在两个对象的投影不重叠的线（称为轴），则两个凸对象不重叠。

无论维度如何，分离轴始终是一条线。例如，在 **3D** 中，空间由平面分隔，但分离轴垂直于分隔平面。

每条面的法向量或其他特征向量将会作为分离轴进行检测。

在 **3D** 中，只使用面法向量无法区分一些边对边的非碰撞情况。因此需要引入额外的轴，在每个物体中取一条边，将两条边的叉积作为分离轴进行检测[^2]。

!!! note "笔记"

    简单的来说，就是尽可能的寻找所有的分离轴去证明这两个物体是分开的，否则两个物体就是相交的。

    在 **2D** 中，每条边的法向量作为分离轴即可。
    
    在 **3D** 中，以面法向量作为分离轴检测，但是需要引入额外的分离轴去判断特殊情况（边碰边）。




*[SAT]: Separating Axis Theorem
*[凸集分离定理]: Hyperplane separation theorem

[^1]:
    [Wiki](https://en.wikipedia.org/wiki/Hyperplane_separation_theorem#Use_in_collision_detection)
[^2]:
    [高级向量教学](https://docs.godotengine.org/en/stable/tutorials/math/vectors_advanced.html#collision-detection-in-3d)