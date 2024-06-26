---
title: 奇异值分解SVD
published: 2023-07-01
description: 奇异值分解SVD的基本概念
tags: [数学]
category: 数学
draft: false
---

## 特征值分解  
回顾特征值分解，对于方阵 A，可求解特征值 $\lambda$ 和特征向量 $\vec x$.，有 $\lambda \vec x = A \vec x$。将所有特征向量以列向量列成矩阵 $W$，则有 $\Sigma W=AW$，其中 $\Sigma$ 是以全部特征值为对角元素的对角矩阵。  
则有特征值分解：  
$$  
A = W\Sigma W^{-1}  
$$  
一般把特征向量全部标准化，已知特征向量两两正交，则特征向量标准化后，形成一组正交基，则 $W$ 是正交矩阵，有：  
$$  
A = W\Sigma W^T  
$$  
  
## 奇异值分解  
特征值分解只对方阵有效，奇异值分解将其“推广”到普通矩阵。比如对于m\*n的矩阵A，分解形式为  
$$  
A=U\Sigma V^T  
$$  
其中，U是 m\*m，$\Sigma$ 是m\*n的对角矩阵，对角线上的值就是奇异值。V是 n\*n 矩阵。  
其中，U是左奇异向量组成的矩阵，根据它的shape，可以很容易记住 U 是 $AA^T$ 的特征向量 $u_i$ 组成的矩阵。V是右奇异向量组成的矩阵，根据它的shape，也可以很容易记住 V 是 $A^TA$ 的特征向量 $v_i$ 组成的矩阵。**做约定，让UV都是正交矩阵（特征向量经过标准化）**  
求 $\Sigma$ 则是求奇异值 $\sigma$ 的过程。  
$$  
\begin{aligned}  
AV&=U\Sigma V^TV \\  
AV&=U\Sigma \\  
Av_i&=\sigma_iu_i \\  
\sigma_i &= \frac{Av_i}{u_i}  
\end{aligned}  
$$  
这是*求法1*. 另外有：  
$$  
\begin{aligned}  
AA^T&=U\Sigma V^TV\Sigma^TU^T \\  
AA^T&=U\Sigma \Sigma^T U^T \\  
\end{aligned}  
$$  
其中 $\Sigma \Sigma^T$ 是 m\*m 矩阵，对角线值为奇异值的平方，而对 $AA^T$ 特征值分解：$AA^T=U\Sigma 'U^T$ ，$\Sigma'$正是 $AA^T$ 特征值组成的对角阵，所以*奇异值是左特征值的开平方*。  
同样，上面的推导过程用 $A^TA$ 也有类似结果，对应右特征值的开平方，用哪个，主要看shape。奇异值数量是 m,n 中较小的那个，看左右那个是 min{m,n} 维方阵。  
  
## 奇异值性质  
奇异值一般下降得非常快，只有前几个奇异值比较大，后面可能都非常接近零。所以它常常用来做特征降维PCA（要求$X^TX$的前几大的特征值和对应的特征矩阵）等。有一些巧妙的奇异值分解求解办法可以在不计算 $A^TA$ 的条件下求出右奇异矩阵，在一些应用中能提供很大的简化。  
  
此外，还有 *Moore-Penrose 伪逆*。它是求逆运算在非方阵上的推广。任意矩阵A的伪逆 定义为：  
$$  
A^+=\lim_{\alpha\to0}(A^TA+\alpha I)^{-1}A^T  
$$  
一般实现用的是奇异值分解给出的近似：  
$$  
A^+ = V\Sigma^+ U^T  
$$  
其中，U, V 分别是 A 的左奇异矩阵和右奇异矩阵。奇异值矩阵（对角阵）的伪逆通过 矩阵转置并对对角值求倒数得到。  
  
对于线性方程组 $Ax=b$，用伪逆去求解 $x=A^+b$。如果方程个数少于未知数个数，则它能求出其中一个解，并且这个解的 L2 范数在所有解中最小；如果方程个数多于未知数个数（可能无解），它可以求出最小二乘解（$||Ax-b||_2$最小）。  
