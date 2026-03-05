# Eigen库的基本使用
Eigen是一个基于C++模板的线性代数库，直接将库下载后放到项目目录下，然后包含头文件就能使用，十分方便。
## Eigen矩阵定义
Eigen以矩阵为基本数据单元。它是一个模板类。它的前3个参数为：数据类型，行，列。
```C++
Eigen::Matrix<float, 2, 3> matrix_23;   // 声明一个2x3的float矩阵
```
同时，Eigen通过typedef提供了许多内置类型，不过底层仍是Eigen::Matrix。   
```C++
#include <Eigen/Dense>

Eigen::Matrix<double, 3, 3> A;               // Fixed rows and cols. Same as Matrix3d.
Eigen::Matrix<double, 3, Dynamic> B;         // Fixed rows, dynamic cols.
Eigen::Matrix<double, Dynamic, Dynamic> C;   // Full dynamic. Same as MatrixXd.
Eigen::Matrix<double, 3, 3, RowMajor> E;     // Row major; default is column-major.
Eigen::Matrix3f P, Q, R;                     // 3x3 float matrix.
Eigen::Vector3f x, y, z;                     // 3x1 float matrix.
Eigen::RowVector3f a, b, c;                  // 1x3 float matrix.
Eigen::VectorXd v;                           // Dynamic column vector of doubles
```

## Eigen基础使用
```C++
x.size();   // vector size
C.rows();   // number of rows
C.cols();   // number of columns

A.resize(4, 4);   // Runtime error if assertions are on.
B.resize(4, 9);   // Runtime error if assertions are on.
A.resize(3, 3);   // Ok; size didn't change.
B.resize(3, 9);   // Ok; only dynamic cols changed.

A << 1, 2, 3,     // Initialize A. The elements can also be
     4, 5, 6,     // matrices, which are stacked along cols
     7, 8, 9;     // and then the rows are stacked.
B << A, A, A;     // B is three horizontally stacked A's.
A.fill(10);       // Fill A with all 10's.
```

## 一些矩阵运算
```C++
// 四则运算就不展示了，直接用对应的运算符即可
Eigen::Matrix3d matrix_33 = Eigen::Matrix3d::Random();
matrix_33.transpose();      // 转置
matrix_33.sum();            // 各元素和
matrix_33.trace();          // 迹
10 * matrix_33;             // 数乘
matrix_33.inverse();        // 逆
matrix_33.determinant();    // 行列式

// 特征值/特征向量
// 实对称矩阵可以保证对角化成功
Eigen::SelfAdjointEigenSolver<Eigen::Matrix3d> eigen_solver (matrix_33.transpose()*matrix_33);
cout << "Eigen values = " << eigen_solver.eigenvalues() << endl;
cout << "Eigen vectors = " << eigen_solver.eigenvectors() << endl;
```