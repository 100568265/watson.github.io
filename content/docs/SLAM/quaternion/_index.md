---
title: "4.2.四元数：旋转的优雅解法"
weight: 2
bookCollapseSection: true
math: true
---

# 四元数 (Quaternion)

## 1.什么是四元数？

四元数是由数学家William Rowan Hamilton于1843年提出的一种数学工具，它将复数从二维推广到了四维空间。一个四元数由**一个实部**和**三个虚部**组成：

$$q = w + xi + yj + zk$$

其中 $i^2 = j^2 = k^2 = ijk = -1$，$w$ 是实部，$x, y, z$ 是虚部。在旋转表示中，通常写成：

$$q = [w, \mathbf{v}] = [w, x, y, z]$$

---



## 2.四元数的作用

四元数（Quaternion）在 SLAM 和机器人领域中主要用于表示三维旋转，是姿态（orientation）表达的核心工具之一。机器人的位姿通常表示为「平移 + 旋转」。旋转部分几乎都用四元数来参数化。

**在 SLAM 中的具体应用场景**

- **状态向量**：EKF-SLAM、图优化 SLAM 中，每个关键帧的旋转状态就是一个单位四元数。 
- **图优化**：在 g2o、Ceres 等优化框架中，位姿节点的旋转部分用四元数（或等价的李群 SO(3)）表达，优化时对其施加局部扰动来求解。
- **IMU 预积分**：视觉惯性 SLAM（如 VINS-Mono、ORB-SLAM3）中，IMU 的角速度积分直接用四元数微分方程来更新姿态。
- **传感器融合**：将来自不同传感器（相机、IMU、轮式里程计）的旋转统一到一个四元数表示下进行融合。

---



## 3. 复数与2D旋转

四元数的表示和计算十分复杂和晦涩，例如四元数为何是这样去表示**3D旋转**的？想要搞清楚四元数背后的原理，我们不妨先从理解**复数和2D旋转**开始下手：



### 复数表达

表达复数使用$z=a+bi$

其中$a,b∈\mathbb{R}, i^2=-1$

我们将a称为这个复数的实部(Real Part)，b称为这个复数的虚部(Imaginary Part)。

用向量来表示一个复数$z=\begin{bmatrix} a \\\ b \end{bmatrix}$

<img src="2dvector.png" alt="image-20260318171409356" style="zoom:50%;" />

用矩阵来表示一个复数$z=\begin{bmatrix} a&-b \\\ b&a \end{bmatrix}$

<details>
<summary>点击展开：为什么可以这样写？</summary>
核心逻辑：i 的矩阵化身
复数最核心的定义是$i^2=-1$，假设我们要找一个矩阵M来代表i：

复数i对应矩阵形式就是$M=\begin{bmatrix} 0&-1 \\\ 1&0 \end{bmatrix}$（单位矩阵I逆时针旋转90度得到的结果）

那么对应的$z=a+bi=\begin{bmatrix} a&0 \\\ 0&a \end{bmatrix} + \begin{bmatrix} 0&-b \\\ b&0 \end{bmatrix} = \begin{bmatrix} a&-b \\\ b&a \end{bmatrix}$

</details>



### 模长与共轭

**复数的模长**：$||z||=\sqrt{a^2+b^2}$

**复数的共轭**：$\bar{z}=a-bi$

$z\bar{z} = (a+bi)(a-bi) = a^2 + b^2$

---

### 复数相乘与2D旋转

<img src="2drotate.png" alt="image-20260318224152198" style="zoom:50%;" />

$\begin{bmatrix} a&-b \\\ b&a \end{bmatrix}=\sqrt{a^2 + b^2}\begin{bmatrix} \frac{a}{\sqrt{a^2 + b^2}}&\frac{-b}{\sqrt{a^2 + b^2}} \\\ \frac{b}{\sqrt{a^2 + b^2}}&\frac{a}{\sqrt{a^2 + b^2}} \end{bmatrix}$

根据三角函数可得：

$\begin{bmatrix} a&-b \\\ b&a \end{bmatrix}=\sqrt{a^2 + b^2}\begin{bmatrix} cos\theta&-sin\theta \\\ sin\theta&cos\theta\end{bmatrix}$

$\begin{bmatrix} a&-b \\\ b&a \end{bmatrix}=||z||\begin{bmatrix} cos\theta&-sin\theta \\\ sin\theta&cos\theta\end{bmatrix}$

$\begin{bmatrix} a&-b \\\ b&a \end{bmatrix}=\begin{bmatrix}||z||&0 \\\ 0&||z||\end{bmatrix} \begin{bmatrix} cos\theta&-sin\theta \\\ sin\theta&cos\theta\end{bmatrix}$

把左边视为缩放矩阵，右边为旋转矩阵

复数的相乘其实是旋转与缩放的复合

---



### 复数的极坐标型

欧拉公式(Euler's Formula)：$cos(\theta)+isin(\theta)=e^{i\theta}$

$z=||z||\begin{bmatrix} cos\theta&-sin\theta \\\ sin\theta&cos\theta\end{bmatrix}$

$z=||z||(cos(\theta)+isin(\theta))$

$z=||z||e^{i\theta}$

如果我们定义r=||z||，我们就得到了复数的极坐标形式：

$z=re^{i\theta}$

我们可以使用一个缩放因子r和旋转角度$\theta$的形式来定义任意一个复数，而且它旋转与缩放的性质仍然存在。

对空间2D向量旋转也就可以表示为：

$v\'=re^{i\theta}v$

---



## 4.轴角表示3D旋转

表示三维空间中旋转的方式有很多种，比如前面介绍过的欧拉角，但欧拉角可能会遇到万向节死锁的问题。

这里我们介绍另一种方式，**轴角**。

<img src="axis-angle.png" alt="image-20260318235059812" style="zoom: 33%;" />

假设我们有一个经过原点的旋转轴$u=(x,y,z)^T$，我们把向量v，沿着旋转轴旋转θ度，变换到$v\'$(坐标系使用的都是右手坐标系)。

使用轴角，潜在是四个变量，旋转轴的(x,y,z)三个变量，以及一个旋转角θ。

实际上我们使用$u=(x,y,z)^T$，这里的**单位向量就是旋转的方向，模长就是旋转角**



### 旋转的分解

我们可以将v分解为平行于旋转轴u以及正交于u的两个分量

$\mathbf{v} = \mathbf{v}\_{\parallel} + \mathbf{v}_{\perp}$

<img src="decomp.png" alt="image-20260319000905815" style="zoom: 33%;" />



根据正交投影的公式，我们可以得出：

$\mathbf{v}\_{\parallel}=\frac{u \cdot v}{u \cdot u}u=\frac{u \cdot v}{||u||^2}u=(u \cdot v)u$ 

因为$\mathbf{v}=\mathbf{v}\_{\parallel}+ \mathbf{v}_{\perp}=\mathbf{v}-(u \cdot \mathbf{v})u$



#### $\mathbf{v}\_{\parallel}$的旋转

可以看出来，$\mathbf{v}\_{\parallel}$因为与u重合，没有被旋转

所以$\mathbf{v\'}\_{\parallel}=\mathbf{v}\_{\parallel}$





#### $\mathbf{v}\_{\perp}$的旋转

<img src="vperprotate.png" alt="image-20260319013825582" style="zoom:80%;" />

我们构造一个同时正交于u和$\mathbf{v}\_{\perp}的向量\omega$

$\omega=u \times \mathbf{v}\_{\perp}$

可以发现这个新的向量$\omega$指向$\mathbf{v}\_{\perp}$逆时针旋转$\frac{\pi}{2}$后的方向，并且和$\mathbf{v}\_{\perp}$一样也处于正交于u的平面内。

$||\omega||=||u \times \mathbf{v}\_{\perp}||=||u|| \cdot \mathbf{v}\_{\perp} \cdot sin(\pi/2)=||\mathbf{v}\_{\perp}||$

所以\$\omega$和$\mathbf{v}\_{\perp}$的模长是相同的，$\omega$也位于圆上

我们让$\omega$和$\mathbf{v}\_{\perp}$构成一个新的直角坐标系，把$\mathbf{v\'}\_{\perp}$进行分解

$\mathbf{v\'}\_{\perp}=\mathbf{v\'_v}+\mathbf{v\'_w}=cos(\theta)\mathbf{v}\_{\perp}+sin(\theta)\omega=cos(\theta)\mathbf{v}\_{\perp}+sin(\theta)(u \times \mathbf{v}\_{\perp})$

**由此我们可以得到：**

$\mathbf{v\'}\_{\perp}=cos(\theta)\mathbf{v}\_{\perp}+sin(\theta)(u \times \mathbf{v}\_{\perp})$



### 罗德里格斯旋转公式Rodrigues' Rotation Formula

将上面两个结果组合就可以获得：

3D空间中任意一个$\mathbf{v\'}$沿着单位向量u旋转θ角度之后的$\mathbf{v\'}$为：

$$ \mathbf{v\'}= \mathbf{v}\cos\theta + (\mathbf{u} \times \mathbf{v})\sin\theta + \mathbf{u}(\mathbf{u} \cdot \mathbf{v})(1 - \cos\theta) $$


---



## 5.四元数

$q = a + bi + cj + dk，(a,b,c,d∈\mathbb{R})$

其中，$i^2=j^2=k^2=ijk=-1$

写成向量$q=\begin{bmatrix} a \\\ b \\\ c \\\ d \end{bmatrix}$

分解成实部虚部$q=[s,v]，(v=\begin{bmatrix} x \\\ y \\\ z \end{bmatrix},s,x,y,z∈\mathbb{R})$


### 运算定义
定义只是类比复数进行衍生定义的结果，几何方式理解比较复杂，暂时只需要记住定义即可。

#### 模长

$||q||=\sqrt{a^2+b^2+c^2+d^2}$

$||q||=\sqrt{s^2+||v||^2}=\sqrt{s^2+vv}，(v \cdot v=||v||^2)$



#### 加减法

实部加减实部，虚部加减虚部，没什么好说的。



#### 标量乘法

$sq=s(a + bi + cj + dk) = sa + sbi +scj + sdk$

注意：四元数的标量乘法是遵守**交换律**的，也就是sq = qs



#### 四元数乘法

四元数的乘法**不遵守交换律**，一般情况下$q_1q_2≠q_2q_1$

$$ \begin{aligned} q_1 q_2 &= (a + b\mathbf{i} + c\mathbf{j} + d\mathbf{k})(e + f\mathbf{i} + g\mathbf{j} + h\mathbf{k}) \\\        &= ae + af\mathbf{i} + ag\mathbf{j} + ah\mathbf{k} \\\        &\quad + be\mathbf{i} + bf\mathbf{i}^2 + bg\mathbf{ij} + bh\mathbf{ik} \\\        &\quad + ce\mathbf{j} + cf\mathbf{ji} + cg\mathbf{j}^2 + ch\mathbf{jk} \\\        &\quad + de\mathbf{k} + df\mathbf{ki} + dg\mathbf{kj} + dh\mathbf{k}^2 \\\       &= ae - bf - cg - dh \\\        &\quad + (af + be + ch - dg)\mathbf{i} \\\        &\quad + (ag + ce + df - bh)\mathbf{j} \\\        &\quad + (ah + de + bg - cf)\mathbf{k} \end{aligned} $$

---



## 6.四元数表示3D旋转

我们可以用四元数表达对一个点的旋转。假设一个空间三维点$p=[x,y,z∈\mathbb{R}^3]$，以及一个由轴角n,θ指定的旋转，那么这个旋转的四元数形式为：

$$ \begin{equation} \begin{split} q = \left[ \cos\frac{\theta}{2},\ n_x\sin\frac{\theta}{2},\ n_y\sin\frac{\theta}{2},\ n_z\sin\frac{\theta}{2} \right]^T \end{split} \end{equation} $$



反之，我们亦可以从单位四元数中算出对应旋转轴与夹角

$$ \begin{equation} \begin{split} \begin{cases} \theta = 2\cos^{-1} q_0  \\\  [n_x, n_y, n_z]^T = \dfrac{[q_1, q_2, q_3]^T}{\sin\frac{\theta}{2}} \end{cases} \end{split} \end{equation} $$



三维点p经过旋转之后变为$p\'$。如果使用矩阵描述，那么有$p\'=Rp$。如果用四元数描述旋转，它们的关系如何来表达？

首先，把三维空间点用一个虚四元数来描述：

$$p=[0,x,y,z]=[0,\mathbf{v}]$$



这相当于我们把**四元数的三个虚部与空间中的三个轴**相对应。然后，参照上面的公式，用四元数q表示这个旋转：

$$q=[cos\frac{\theta}{2},nsin\frac{\theta}{2}]$$



那么，旋转后的点$p\'$即可表示为这样的乘积：

$$p\'=qpq^{-1}$$



### 四元数到旋转矩阵的转换

任意单位四元数描述了一个旋转，该旋转也可以用旋转矩阵或旋转向量描述。从旋转向量到四元数的方式已经在式(2)中给出。现在直接给出四元数到旋转矩阵的转换方式。

设四元数$q=q_0+q_1i+q_2j+q_3k$，对应的旋转矩阵$\mathbf{R}$为：

$$ \begin{equation} \begin{split} R =  \begin{bmatrix} 1 - 2q_2^2 - 2q_3^2 & 2q_1 q_2 + 2q_0 q_3 & 2q_1 q_3 - 2q_0 q_2 \\\ 2q_1 q_2 - 2q_0 q_3 & 1 - 2q_1^2 - 2q_3^2 & 2q_2 q_3 + 2q_0 q_1 \\\ 2q_1 q_3 + 2q_0 q_2 & 2q_2 q_3 - 2q_0 q_1 & 1 - 2q_1^2 - 2q_2^2 \end{bmatrix} \end{split} \end{equation} $$



反之，由旋转矩阵到四元数的转换如下。假设矩阵为$\mathbf{R}={m_{ij},i,j∈ [1,2,3]}$，其对应的四元数$\mathbf{q}$由下式给出：

$$ \begin{equation} \begin{split} q_0 &= \frac{\sqrt{\operatorname{tr}(R) + 1}}{2},\quad q_1 = \frac{m_{23} - m_{32}}{4q_0},\quad q_2 = \frac{m_{31} - m_{13}}{4q_0},\quad q_3 = \frac{m_{12} - m_{21}}{4q_0} \end{split} \end{equation} $$

---



## 7.实践：Eigen几何模块

| 数学命名              | Eigen库结构           |
| --------------------- | --------------------- |
| 旋转矩阵（3 x 3）     | `Eigen::Matrix3d`     |
| 旋转向量（3 x 1）     | `Eigen::AngleAxisd`   |
| 欧拉角（3 x 1）       | `Eigen::Vector3d`     |
| 四元数（4 x 1）       | `Eigen::Quaterniond`  |
| 欧氏变换矩阵（4 x 4） | `Eigen::Isometry3d`   |
| 仿射变换（4 x 4）     | `Eigen::Affine3d`     |
| 射影变换（4 x 4）     | `Eigen::Projective3d` |



```cpp
// 引入Eigen核心模块（矩阵、向量）和几何模块（变换、旋转）
#include <Eigen/Core>
#include <Eigen/Geometry>
#include <iostream>
#include <cmath>

using namespace std;
```



**1.旋转向量 & 旋转矩阵**

```cpp
// 旋转矩阵（3×3）：初始化为单位矩阵（无旋转状态）
Eigen::Matrix3d rotation_matrix = Eigen::Matrix3d::Identity();

// 旋转向量（角轴 AngleAxis）：绕Z轴旋转45°（M_PI/4弧度）
// 参数1：旋转角度（弧度），参数2：旋转轴单位向量
Eigen::AngleAxisd rotation_vector(M_PI/4, Eigen::Vector3d(0,0,1));
// 设置输出精度（保留3位小数，更易阅读）
cout.precision(3);
cout << "旋转矩阵 =\n" << rotation_vector.matrix() << endl;
    
// 旋转向量 → 旋转矩阵（通过toRotationMatrix()接口转换）
rotation_matrix = rotation_vector.toRotationMatrix();
```





**2.旋转3D点**

```cpp
Eigen::Vector3d v(1,0,0);	// 定义一个原始3D点

// 方法1：用旋转向量旋转点（Eigen重载了*运算符，直接计算）
Eigen::Vector3d v_rotated = rotation_vector * v;
cout << "(1,0,0) 经旋转向量旋转后 = " << v_rotated.transpose() << endl;

// 方法2：用旋转矩阵旋转点（数学上就是 R * v）
v_rotated = rotation_matrix * v;
cout << "(1,0,0) 经旋转矩阵旋转后 = " << v_rotated.transpose() << endl;
```



**3.旋转矩阵&欧拉角**

```cpp
// 旋转矩阵 → 欧拉角（ZYX顺序，对应 yaw(偏航)、pitch(俯仰)、roll(滚转)）
// eulerAngles(2,1,0) 对应 Z(2)、Y(1)、X(0) 轴顺序
Eigen::Vector3d euler_angles = rotation_matrix.eulerAngles(2,1,0); 
cout << "yaw(偏航) pitch(俯仰) roll(滚转) = " << euler_angles.transpose() << endl;
```





**4.欧式变换矩阵（旋转+平移）**

`Eigen::Isometry3d` 是欧式变换的专用类，本质是4×4齐次矩阵

```cpp
Eigen::Isometry3d T = Eigen::Isometry3d::Identity();    // 初始化为单位矩阵（无旋转、无平移）
T.rotate(rotation_vector);  // 给欧式变换设置旋转（用之前定义的旋转向量）
T.pretranslate(Eigen::Vector3d(1,3,4)); // 设置平移向量 t = (1, 3, 4)
cout << "欧式变换矩阵 T =\n" << T.matrix() << endl;
```





**5.用欧式变换矩阵变换3D点**

```cpp
// 对原始点v执行欧式变换：p' = R*p + t（Eigen自动处理齐次坐标，直接用*运算）
Eigen::Vector3d v_transformed = T*v;  
cout << "v 经欧式变换后 = " << v_transformed.transpose() << endl;
```



**6.四元数旋转3D点（最优旋转表示）**

```cpp
// 旋转向量 → 四元数（直接赋值即可，Eigen自动转换）
Eigen::Quaterniond q = Eigen::Quaterniond(rotation_vector);
cout << "四元数 q = \n" << q.coeffs() << endl; // 输出顺序：(x, y, z, w)，w为实部

// 用四元数旋转3D点（数学原理：p' = q*p*q⁻¹，Eigen重载*运算符，直接调用）
v_rotated = q*v;  
cout << "(1,0,0) 经四元数旋转后 = " << v_rotated.transpose() << endl;
```





<details><summary>完整代码</summary>

```cpp
#include <iostream>
#include <cmath>
using namespace std;

#include <Eigen/Core>
#include <Eigen/Geometry>

int main(){
    // 3D旋转矩阵直接使用Matrix3d或Matrix3f
    Eigen::Matrix3d rotation_matrix = Eigen::Matrix3d::Identity();
    std::cout << "!" << std::endl;
    // 旋转向量使用AngleAxis，底层不直接是Matrix，但运算可以当作矩阵(运算符重载)
    Eigen::AngleAxisd rotation_vector(M_PI/4,Eigen::Vector3d(0,0,1));   //沿着Z轴旋转45度
    cout.precision(3);
    cout << "rotation matrix =\n" << rotation_vector.matrix() << endl;
    // 也可以这样
    rotation_matrix = rotation_vector.toRotationMatrix();

    // 用AngleAxis可以进行坐标变换
    Eigen::Vector3d v(1,0,0);
    Eigen::Vector3d v_rotated = rotation_vector * v;
    cout << "(1,0,0) after rotation = " << v_rotated.transpose() << endl;

    // 或者用旋转矩阵
    v_rotated = rotation_matrix * v;
    cout << "(1,0,0) after rotation = " << v_rotated.transpose() << endl;

    // 欧拉角 <-> 旋转矩阵
    Eigen::Vector3d euler_angles = rotation_matrix.eulerAngles(2,1,0); // ZYX顺序，yaw pitch roll
    cout << "yaw pitch roll = " << euler_angles.transpose() << endl;

    // 欧氏变换矩阵使用 Eigen::Isometry
    Eigen::Isometry3d T = Eigen::Isometry3d::Identity();    //欧式变换的专用矩阵
    T.rotate(rotation_vector);  // 按照rotation_vector进行旋转
    T.pretranslate(Eigen::Vector3d(1,3,4)); // 把平移向量设成(1,3,4)
    cout << "Transform matrix = \n" << T.matrix() << endl;

    // 用变换矩阵进行坐标变换
    Eigen::Vector3d v_transformed = T*v;    // p′ = Rp+t
    cout << "v transformed = " << v_transformed.transpose() << endl;

    // 四元数
    // 可以直接把AngleAxis赋值给四元数，反之亦然
    Eigen::Quaterniond q = Eigen::Quaterniond(rotation_vector);
    cout << "quaternion = \n" << q.coeffs() << endl; // (x,y,z,w),w为实部

    // 使用四元数旋转一个向量，使用重载的乘法即可
    v_rotated = q*v;    // 数学上是qvq^{-1}
    cout << "(1,0,0) after rotation = " << v_rotated.transpose() << endl;

}
```

</details>



## 8.SLAM相关

实际当中，我们至少定义两个坐标系：**世界坐标系**和**相机坐标系**。在该定义下，设某个点在世界坐标系中坐标$\mathbf{p_\omega}$，在相机坐标系下为$\mathbf{p_c}$，那么：

$$\mathbf{p_c}=T_{c\omega}\mathbf{p_\omega}$$

这里$T_{c\omega}$表示世界坐标系到相机坐标系间的变换。或者我们可以用反过来的$ T_{c\omega} $：

$$ \mathbf{p_\omega} = T_{\omega c}\mathbf{p_c}=T^{-1}_{c\omega}\mathbf{p_c}$$



如果把上面两式的$\mathbf{p_c}$取成零向量，也就是相机坐标系中的原点，那么，此时的$\mathbf{p_\omega}$就是相机原点在世界坐标系下的坐标：

$$\mathbf{p_\omega}=T_{\omega c}0=t_{\omega c}$$
