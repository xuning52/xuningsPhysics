求解线性方程组
返回近似解
#
输入
nxn矩阵A厄密矩阵
（构造厄密矩阵：令c=$\begin{pmatrix}0&A\\ A^{\dagger}&0\end{pmatrix}$，求解Cy=$\begin{pmatrix}b\\0\end{pmatrix}$，得y=$\begin{pmatrix}0\\x \end{pmatrix}$）
输入b单位向量

对矩阵A实空间扩展：
### 1. 扩展原理：复数的拆解

复数 $z = a + bi$ 可以映射为一个 $2 \times 2$ 的实矩阵：

$$z \to \begin{pmatrix} a & -b \\ b & a \end{pmatrix}$$

同理，对于一个复矩阵中的每个元素 $z_{jk} = a_{jk} + i b_{jk}$，我们都可以用这个 $2 \times 2$ 的结构去替换它。

### 2. 对矩阵 A 进行实空间扩展

你的矩阵是：

$$A = \begin{pmatrix} 0 & -i \\ i & 0 \end{pmatrix}$$

我们把每个位置的元素进行替换：

- **左上角 $0$ (即 $0 + 0i$)**：
    
    变为 $\begin{pmatrix} 0 & 0 \\ 0 & 0 \end{pmatrix}$
    
- **右上角 $-i$ (即 $0 - 1i$)**：
    
    这里 $a=0, b=-1$，带入 $\begin{pmatrix} a & -b \\ b & a \end{pmatrix}$ 得到 $\begin{pmatrix} 0 & 1 \\ -1 & 0 \end{pmatrix}$
    
- **左下角 $i$ (即 $0 + 1i$)**：
    
    这里 $a=0, b=1$，得到 $\begin{pmatrix} 0 & -1 \\ 1 & 0 \end{pmatrix}$
    
- **右下角 $0$**：
    
    变为 $\begin{pmatrix} 0 & 0 \\ 0 & 0 \end{pmatrix}$

合并得到4x4矩阵。
共轭矩阵的等价实数矩阵是对称矩阵。