[TOC]

# 一、信号与系统

信号可以来自于很多外部设备：录音机、温度计、相机

**信号处理的方向**：并不关心数据以怎样自然规律产生，而是重点放在如何改变输入的信号数据

- 模拟信号：连续不断的，针对具体外界做出的具体测量值，没有取值范围
- 数字信号：离散采样的，具有时间和幅度两个分量，将模拟信号限定在一个**固定**的测量范围后，表示**局部模拟信号**波动的值，在传输时具有较强的抗干扰能力




## 1. 信号的系统

> 信号的系统指一系列处理信号的**方法**，

### 1.1 信号系统的分类

- 线性 Linear / 非线性 Non-linear
- 时不变 time-inveriant / 时变 time-varying



### 1.2 对信号分析和表达的作用域

- 时域：从时间的角度分析信号
- 频率域：从信号频率角度分析信号
- 空间域：从信号发生空间位置角度分析信号



## 2. 信号的分类

> 信号在离散和连续时有相同也有不同的特性，比如：一个正弦信号，如果周期不是整数，那虽然在连续信号里它具有周期性，但是在离散信号里，没有周期性

### 2.1 单位脉冲信号

**脉冲信号**：一种离散信号（即，离散时间信号，指信号在时间这个轴上是离散的），波形之间在 Y 轴不连续，具有一定周期性
![](./images/unit_impulse.png)
$$
\begin{array}{rrl}
Original & \delta(n) =& 
\begin{cases}
1,& n=0\\
0,& otherwise
\end{cases}\\
Offset & \delta(n-n') =& 
\begin{cases}
1,& n=n'\\
0,& otherwise
\end{cases}\\\\
2D Discrete & \delta(n) =& f_1(n_1)\cdot f_2(n_2)
\end{array}
$$



### 2.2 单位跃阶信号

![](./images/unit_step.png)
$$
\begin{array} {rrl}
Original & u(n) =& 
\begin{cases}
1,& n\geq0\\
0,& otherwise
\end{cases}\\
Offset & u(n-n') =& 
\begin{cases}
1,& n \geq n'\\
0,& otherwise
\end{cases}\\\\
2D Discrete & u(n_1, n_2) =& 
u_1(n_1)\cdot u_2(n_2)
\end{array}
$$



### 2.3 正弦信号

A 振幅，$\omega_0$ 频率，$\phi$ 相位（指偏移量），$T_0 = {2 \pi \over \omega_0}$ 周期
$$
u(t) = A \space cos(\omega_0 t + \phi)
$$

![](./images/cos.png)



### 2.4 指数信号

指数信号
$$
u(t) = C e^{at}，(C \space 和 \space a \space 是实数)
$$
![](./images/exponent.png)



复指数信号
$$
u(t) = C e^{at}，(C \space 和 \space a \space 是复数)\\
u(t) = |C|e^{rt}e^{(j\omega_0t+ \theta)}
$$

![](./images/exponent2.png)




# 二、卷积



# 三、傅立叶变换



# 四、直方图







# 引用

1. [正弦信号，指数信号](https://www.cnblogs.com/zhiyinglky/p/5805315.html)
2. [跃阶信号](https://www.cnblogs.com/zhiyinglky/p/5805314.html)
3. [Fundamentals of Digital Image and Video Processing](https://github.com/giosans/Fundamentals-of-Digital-Image-and-Video-Processing-course)
4. [Signals and Systems](http://open.163.com/movie/2011/8/7/A/M8AROL7GG_M8AROT67A.html)