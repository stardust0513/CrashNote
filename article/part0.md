[TOC]

# 一、向量

##1. 概念

- 起点是原点
- 向量 = 方向 + 大小
- **位置移动** 的数值记录
- **线性变换 ** 的数值记录
  ​

##2. 基本运算 

- 向量的加法
  ![](../images/vectorAdd.png)
  ​
- 向量的缩放
  ![](../images/vectorScale.png)
  ​


## 3. 向量的基

- 向量的基 == 单位向量

- 向量的值 **依赖** 于基
  ![](../images/vectorBase.png)

- 向量空间的一个基是 **张成** 该空间的 **线性无关** 向量集

  ​

# 二、张成空间

## 1. 概念

- 二维的张成空间
  ![](../images/span1.png)

- 三维的张成空间
  ![](../images/span2.png)
  ​

## 2. 线性相关

- 线性相关：有多组向量构成张成空间，移除其中一个 **不减少** 张成空间的大小
线性无关： 所有向量 给张成空间增添了新的维度

- 线性相关：多组向量中，有向量 = 其他向量的线性组合
  ![](../images/linear.png)
  ​
  
# 三、线性变换

## 1. 概念

- 实质：操作空间的一种手段

- 原点固定

- 直线在变换后保持直线

- 网格保持 **平行** 且 **等距分布**

  ![](../images/linearTransform.png)
  ​

## 2. 矩阵左乘（列向量矩阵）

- 实质：求出右边的向量 **对应基坐标变化后的值**

- 线性变换后的基坐标 == 矩阵
  ![](../images/LinearTransform1.png)

- 矩阵 * 向量 == 该向量对应 **基坐标的线性变换** 后的值

  ![](../images/LinearTransform2.png)



## 3. 矩阵相乘

- 实质：先后切换基坐标的状态
  例子：矩阵B * 矩阵A * 向量C == 将向量C **先按照 矩阵A 变换**，再按照矩阵B 变换
  ![](../images/matrix.png)

- 矩阵相乘先后次序不同，变换结果不同

  例子：矩阵B * **矩阵A** != **矩阵A** * 矩阵B

- 矩阵相乘结合律：(AB) C == A (BC)
  由于**都是从 最右边的 C 开始变换**，从几何角度看没有区别

- 矩阵相乘公式
  ![](../images/matrix1.png)



# 四、行列式

## 1. 概念

- 表示：区域 **面积** 线性变换后增大和减小的 **比例**
  **符号取决于** 区域是否发生了翻转变换（翻转为 负）
  ![](../images/determinant.png)

- 表示：区域 **体积** 线性变换后增大和减小的 **比例**
  **符号取决于** 构成矩阵的这三个向量**是否满足右手定则**（不满足为 负）
  ![](../images/determinant3.png)
## 2. 特殊情况

- 当线性变化后 区域 **面积** 为 0 时（点，线）
  ![](../images/determinant1.png)
  
- 当线性变化后 区域 **体积** 为 0 时（点，线，面）
  ![](../images/determinant4.png)

- 当线性变化后 比例为 **负数**  时，平面翻转
  ![](../images/determinant2.png)
  
## 3. 公式

- ![](../images/determinant5.png)
  ​

#五、矩阵和线性方程组

## 1. 线性方程组 转 矩阵

- 求线性方程组的解，就是求线性变化的向量
 ![](../images/equation.png)

## 2. 列空间

- **零向量** 一定在列空间中（因为线性变换必须保持原点不变）
 ![](../images/columSpace.png)

## 3. 秩

- 实质：线性变换后空间的 **维数**
- 定义：列空间的维数
- 满秩：秩 == 列数

## 4. 零空间

- 当一个矩阵不是满秩的时候，会出现零空间
- 实质：线性变换后落在原点的集合
- 当线性方程组线性变换前 **V 向量为 0 时**，零空间就是该线性方程组 **解的集合**

## 5. 逆矩阵

- 前提：矩阵的行列式的值不为 0
- 可以利用矩阵的逆阵求解方程组。
- 实质：矩阵A 线性变换后，在变换矩阵A 的逆矩阵，向量的基不变。且无论先后顺序，向量的基都不变。（恒等变换）
   ![](../images/inverseMatrix.png)
   
## 6. 非方阵
- 3 X 2 矩阵的几何意义：将 **二维** 空间映射到 **三维**空间上


# 六、向量的点积

##1. 标准
- 数学计算，遵循乘法交换律（都是长度乘长度）
  ​
  $$
  \begin{bmatrix} 
  \color{green}{1} \\  \color{red}{2} \\ 
  \end{bmatrix} \cdot
  \begin{bmatrix} 
  \color{green}{3} \\  \color{red}{4} \\ 
  \end{bmatrix} = \color{green}{1} \cdot \color{green}{3} + \color{red}{2} \cdot \color{red}{4}
  $$

- 几何解释
V * W > 0 方向相同
V * W < 0 方向反相
V * W = 0 互相垂直
  **W** 到 V 的投影的长度 * V 的长度
  ![](../images/dot1.png)
  
##2. 通过线性变换理解点积
1. 二维向量 线性变换到 一维向量

  `基向量 X: (0, 1) -> 1`
  `基向量 Y: (1, 0) -> -2`

  ![](../images/dot2.png)

2. 利用对称性，二维平面上一个向量 u，变换到 一维的一条线上
  ![](../images/dot4.png)

3. 上述 1- 2 情况下，矩阵A * 向量B == 向量C * 向量B
  向量B 和 向量C 的**点积的值**可以用 向量B 到 C 的投影值 和 向量 C 的值的积求出
  ![](../images/dot3.png)

4. 综上，矩阵A 这样的由二维到一维的线性变换过程 可以用 向量C 表示，并且 **矩阵A 和 向量C** 一一对一
  ![](../images/dot5.png)


# 七、向量的叉乘

## 1. 标准

- 叉乘计算：求两个向量组合成的矩阵的行列式值（矩阵的转置矩阵不改变行列式的值）
  ![](../images/cross1.png)

- 几何意义：叉乘的两个向量的面积（叉乘和方向有关， **不遵循交换律**）
  正方向：右边向量 X **左边向量**
  反方向：**左边向量** X 右边向量
  ![](../images/cross.png)

- 实际作用：通过 2 个三维向量生成 1 个**新的三维向量**
**叉乘的结果是一个向量**
  ![](../images/cross2.png)

- 公式
  ​
  $$
  \begin{bmatrix} 
  \color{green}{v_1} \\  \color{red}{v_2} \\ \color{blue}{v_3}\\ 
  \end{bmatrix} \times
  \begin{bmatrix} 
  \color{green}{w_1}  \\ \color{red}{w_2} \\ \color{blue}{w_3} \\ 
  \end{bmatrix} = 
  \begin{bmatrix} 
  \color{red}{v_2} \cdot  \color{blue}{w_3} - \color{blue}{v_3}\cdot \color{red}{w_2} \\
  \color{blue}{v_3} \cdot  \color{green}{w_1} - \color{green}{v_1}\cdot \color{blue}{w_3} \\
  \color{green}{v_1} \cdot  \color{red}{w_2} - \color{red}{v_2}\cdot \color{green}{w_1} \\
  \end{bmatrix}
  \\ \Downarrow \\
  \begin{bmatrix} 
  \color{#F80}{v_1} \\  \color{#F80}{v_2} \\ \color{#F80}{v_3}\\ 
  \end{bmatrix} \times
  \begin{bmatrix} 
  \color{#F0F}{w_1}  \\ \color{#F0F}{w_2} \\ \color{#F0F}{w_3} \\ 
  \end{bmatrix} = 
  det
  \begin{pmatrix} 
  \begin{bmatrix} 
  \color{green}{\hat x}  & \color{#F80}{v_1} & \color{#F0F}{w_1} \\
  \color{red}{\hat y}  & \color{#F80}{v_2} & \color{#F0F}{w_2} \\ 
  \color{blue}{\hat z}  & \color{#F80}{v_3} & \color{#F0F}{w_3} \\ 
  \end{bmatrix} 
  \end{pmatrix}
  \\ \Downarrow \\
  \color{#F80}{\overrightarrow V} \times \color{#F0F}{\overrightarrow W}
  = 
  \color{green}{\hat x}(\color{#F80}{v_2} \cdot  \color{#F0F}{w_3} - \color{#F80}{v_3}\cdot \color{#F0F}{w_2}) + 
  \color{red}{\hat y}(\color{#F80}{v_3} \cdot  \color{#F0F}{w_1} - \color{#F80}{v_1}\cdot \color{#F0F}{w_3}) + 
  \color{blue}{\hat z}(\color{#F80}{v_1} \cdot  \color{#F0F}{w_2} - \color{#F80}{v_2}\cdot \color{#F0F}{w_1})
  $$
  ​

## 2. 通过线性变换理解叉乘

1. 通过第一列的向量的**三个坐标得到一个行列式的值**，构造 三维空间 到 一维空间的**线性变换**（由于，所以是线性变换）关系
   ![](../images/cross5.png)

2. 由于函数线性，就会存在一个 1 X 3 矩阵来代表这个变换
   ​
   $$
   \begin{bmatrix} \color{red}{p_1} &  \color{red}{p_2} & \color{red}{p_3}\\ \end{bmatrix} \cdot 
   \begin{bmatrix} x \\ y \\ z\\ \end{bmatrix} =  
   det
   \begin{pmatrix} 
   \begin{bmatrix} 
   x & \color{green}{v_1} & \color{#F80}{w_1} \\
   y & \color{green}{v_2} & \color{#F80}{w_2} \\ 
   z & \color{green}{v_3} & \color{#F80}{w_3} \\ 
   \end{bmatrix} 
   \end{pmatrix}
   $$



3. 从多维到一维的线性变换矩阵可以用向量代替（见 六.2）
$$
\begin{bmatrix} \color{red}{p_1}\\ \color{red}{p_2}\\ \color{red}{p_3}\\ \end{bmatrix} \cdot 
\begin{bmatrix} x \\ y \\ z\\ \end{bmatrix} =  
det
\begin{pmatrix} 
\begin{bmatrix} 
x & \color{green}{v_1} & \color{#F80}{w_1} \\
y & \color{green}{v_2} & \color{#F80}{w_2} \\ 
z & \color{green}{v_3} & \color{#F80}{w_3} \\ 
\end{bmatrix} 
\end{pmatrix} 
\\ \Downarrow \\
\color{red}{p_1} \cdot x +  \color{red}{p_2} \cdot y +  \color{red}{p_3} \cdot z = x(\color{green}{v_2} \cdot  \color{#F80}{w_3} - \color{green}{v_3}\cdot \color{#F80}{w_2}) + 
 y(\color{green}{v_3} \cdot  \color{#F80}{w_1} - \color{green}{v_1}\cdot \color{#F80}{w_3}) + 
 z(\color{green}{v_1} \cdot  \color{#F80}{w_2} - \color{green}{v_2}\cdot \color{#F80}{w_1})
\\ \Downarrow \\
\color{red}{p_1} = \color{green}{v_2} \cdot  \color{#F80}{w_3} - \color{green}{v_3}\cdot \color{#F80}{w_2} \\
\color{red}{p_2} = \color{green}{v_3} \cdot  \color{#F80}{w_1} - \color{green}{v_1}\cdot \color{#F80}{w_3} \\
\color{red}{p_3} = \color{green}{v_1} \cdot  \color{#F80}{w_2} - \color{green}{v_2}\cdot \color{#F80}{w_1}
$$

4. P * 未知向量  == 未知向量、V、W 确定的平行六面体的体积
   ![](../images/cross3.png)
5. P * 未知向量 == Z 的长度（P 在 z 轴投影）* P 的长度（见 六.1）
6. 联立 4、5 结果可得：
   P 的长度 == V、W 确定的平行四边形面积 == V、W 的叉乘
   ![](../images/cross4.png)

#八、基变换

#九、特征向量与特征值

#十、抽象向量空间