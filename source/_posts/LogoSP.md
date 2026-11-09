---
title: LogoSP
tags: paper blog
mathjax: true
abbrlink: cdbd97ec
date: 2026-11-09 14:54:43
---

Comment: CVPR 2025. Code and data are available at: https://github.com/vLAR-group/LogoSP

# 3.3.1（1）公式推导部分的核心结论

这段内容是**图傅里叶变换（GFT, Graph Fourier Transform）**的标准流程，用到的图论/图信号处理核心概念：**邻接矩阵、度矩阵、归一化图拉普拉斯矩阵、拉普拉斯谱分解、图频域、图基/图频率分量**，是图信号处理（GSP）最经典的一套理论。

下面一步步拆开讲，对应你这段公式：

> $$
> (\boldsymbol{L} = \boldsymbol{D}^{-1/2}(\boldsymbol{D}-\boldsymbol{A})\boldsymbol{D}^{-1/2}) (\boldsymbol{L}=\boldsymbol{U}\boldsymbol{\Lambda}\boldsymbol{U}^\top)
> $$

## 1. 基础图矩阵：

$$
(\boldsymbol{A})
$$

## 邻接矩阵

$$
(\boldsymbol{D})
$$

## 度矩阵

设全局图 G 一共有 S 个节点（文中叫 **superpoint 超点**，是这篇论文自己定义的节点，不是标准图论名词）

- $$
  \boldsymbol{(\boldsymbol{A})}\in\mathbb{R}^{S\times S}
  $$

  ：**邻接矩阵**，描述图里面节点之间有没有边、边权重多大
  $$
  (\boldsymbol{A})_{ij}
  $$
  代表节点i和j的连接权重。

- $$
  \boldsymbol{(\boldsymbol{D})}
  $$

  ：**度矩阵**，对角矩阵。对角元
  $$
  (\boldsymbol{D}){ii}=\sum_j (\boldsymbol{A}){ij}
  $$
  ，就是第i个节点所有相连边权重之和（行和）。

## 2. 归一化图拉普拉斯矩阵

$$
\boldsymbol{(\boldsymbol{L})}
$$

## （重点）

图拉普拉斯有好几种版本，这里是**对称归一化拉普拉斯**，也是GFT里最常用的一种：
$$
\boldsymbol{(\boldsymbol{L})} = \boldsymbol{(\boldsymbol{D})}^{-1/2}(\boldsymbol{(\boldsymbol{D})}-\boldsymbol{(\boldsymbol{A})})\boldsymbol{(\boldsymbol{D})}^{-1/2}
$$

- 未归一化拉普拉斯：
  $$
  \boldsymbol{(\boldsymbol{L})}_{raw}=\boldsymbol{(\boldsymbol{D})}-\boldsymbol{(\boldsymbol{A})}
  $$

- 乘上
  $$
  \boldsymbol{(\boldsymbol{D})}^{-1/2}
  $$
  做归一化，目的：消除节点度数差异带来的偏差，让谱性质更稳定，适合做图傅里叶变换。

- 性质：
  $$
  \boldsymbol{(\boldsymbol{L})}
  $$
  是**实对称半正定矩阵**，所以一定可以正交对角化（谱分解），也就是下一个公式。

## 3. 谱分解 + 图傅里叶变换 GFT【本段最核心知识】

$$
(\boldsymbol{L}=\boldsymbol{U}\boldsymbol{\Lambda}\boldsymbol{U}^\top)
$$

- $$
  {(\boldsymbol{L})}
  $$

  ：对角矩阵，对角线上是**特征值**
  $$
  \lambda_1,\lambda_2,...,\lambda_S
  $$
  ，在GFT里直接对应**图频率**

- $$
  \boldsymbol{(\boldsymbol{U})}
  $$

  ：正交矩阵，每一列
  $$
  \boldsymbol{u}_s
  $$
  是
  $$
  \boldsymbol{(\boldsymbol{L})}
  $$
  的**特征向量** ✅ 在图傅里叶变换的定义里：

  > 拉普拉斯的特征向量
  > $$
  > {\boldsymbol{u}_s}
  > $$
  > 就是**图频域的基函数（basis）**，类比传统傅里叶里的正弦、余弦基！

传统傅里叶：用
$$
\sin/\cos
$$
作为基，把时域信号分解成不同频率分量； 图傅里叶：没有规则网格，用**图拉普拉斯特征向量作为图专属的基**，把定义在图节点上的图信号，分解成不同“图频率”的分量。

- 小特征值 → 低频分量：信号在图上变化平缓、光滑，对应**全局、大尺度模式**
- 大特征值 → 高频分量：信号在图上剧烈跳变，局部变化明显

## 4. 论文里的引申说法（工程/建模层面，不是纯理论定义）

> each eigenvector … can be naturally regarded as a specific global pattern jointly defined by all S superpoints the S eigenvectors in 
> $$
> (\boldsymbol{U})
> $$
> , encompass rich semantic priors

这是作者的建模解读： 因为这些特征向量是整张**全局图**共同解出来的，依赖所有超点之间的连接关系，不是局部信息，所以作者把每一个特征向量解释成一种**全局模式**，这些模式自带图结构带来的先验（semantic priors 语义先验），后续一般用来做图特征、图卷积、全局信息建模。

> ⚠️ 注意：“特征向量=全局模式”是**论文的解释视角**，不是GFT本身的数学定义；数学上它只是图频基函数。

## 5. 用到的知识清单汇总

1. 图表示：邻接矩阵 
   $$
   \boldsymbol{(\boldsymbol{A})}
   $$
   ​                 度矩阵 
   $$
   \boldsymbol{(\boldsymbol{D})}
   $$
   
2. 图拉普拉斯：对称归一化拉普拉斯矩阵

3. 实对称矩阵正交谱分解（线性代数基础，GFT依赖它）

4. 图信号处理（GSP）：图傅里叶变换 GFT

5. 图频率：拉普拉斯特征值对应频率，特征向量对应频域基

## 6.总结

作者先把超点构成全局图，算出归一化拉普拉斯；利用GFT，对拉普拉斯做谱分解，得到一组图频率基；这组基由整张图结构共同决定，因此被用作捕捉全局结构/语义先验的全局模式。
