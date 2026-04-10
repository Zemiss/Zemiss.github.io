---
layout:     post
title:      "MAII-Homework-6"
subtitle:   " \"数学分析没学分qwq\""
date:       2026-04-09 17:50:00
author:     "Zemiss"
header-img: "img/bg-MAII.jpg"
catalog: true
mathjax: true
tags:
  - MAII
  - MA
---

# 数学分析II-第六次作业

## 10.3 连续函数的重要性质

### 第1题

**条件：设 $f(x,y)$ 是 $\mathbb{R}^2$ 上的连续函数，且 $\lim\limits_{\substack{x^2+y^2 \to +\infty}} f(x,y) = +\infty$**

**题目：证明 $f(x,y)$ 在 $\mathbb{R}^2$ 上有最小值。**

**思路：**
1.  **取基准值**：任取一点（如原点），记其函数值为 $C = f(0,0)$。
2.  **用极限圈定有界区域**：由 $\lim\limits_{x^2+y^2\to+\infty}f(x,y)=+\infty$，存在半径 $R>0$，使得圆外所有点的函数值都大于 $C$
3.  **用闭集最值定理**：以原点为圆心、$R$ 为半径的闭圆盘是有界闭集，连续函数 $f$ 在其上必能取到最小值。
4.  **结论**：这个闭圆盘上的最小值，就是全平面 $\mathbb{R}^2$ 上的最小值。

## 10（A）习题

### 第7题

**题目：设$n$为正整数，证明存在$n$阶多项式$p(x)$，使得
$$\max_{-1\leq x\leq 1} \left\vert p(x)\right\vert = \inf_{(a_1,\dots,a_n)\in\mathbb{R}^n} \left\{ \max_{-1\leq x\leq 1} \left\vert x^n + a_1x^{n-1} + \dots + a_{n-1}x + a_n\right\vert \right\}.$$**

**思路：**
1.  **构造范数函数**：定义$g(a') = \max_{-1\leq x\leq1}\left\vert a_0x^n+\dots+a_n\right\vert$，验证它是$\mathbb{R}^{n+1}$上的范数，由有限维空间范数等价性，得$g$连续且$g(a')\geq c\left\vert a'\right\vert$（$c>0$为常数）。
2.  **转化目标函数**：令$a'=(1,a)$（对应首项系数为1的多项式），定义$f(a)=g((1,a))$，则$f$连续，且由范数不等式得$f(a)\to+\infty$当$\left\vert a\right\vert\to+\infty$（强制性质）。
3.  **利用紧集极值定理**：连续函数在$\mathbb{R}^n$的紧集上必取到最小值，因此存在$b\in\mathbb{R}^n$使$f(b)=\inf_{a\in\mathbb{R}^n}f(a)$，对应多项式$p(x)=x^n+b_1x^{n-1}+\dots+b_n$即为所求。

### 第8题

**条件：设$\Omega$是$\mathbb{R}^m$中的区域，$f(X)$定义在$\Omega$上，对于$X_0 \in \Omega$**

*  如果对任给的$\varepsilon > 0$，存在$\delta > 0$，使得当$X \in \Omega$且$\left\vert X - X_0\right\vert < \delta$时，
$$f(X) < f(X_0) + \varepsilon,$$
则称$f(X)$在$X_0$上半连续。

*  如果对任给的$\varepsilon > 0$，存在$\delta > 0$，使得当$X \in \Omega$且$\left\vert X - X_0\right\vert < \delta$时，
$$f(X) > f(X_0) - \varepsilon,$$
则称$f(X)$在$X_0$下半连续。


**题目：设$X_0 \in \Omega$且$X_0$是$\Omega$的聚点**

**(1) $f(X)$在$X_0$上半连续的充要条件是$f(X)$在$X_0$的邻域内有上界且
$$\varlimsup_{\Omega\ni X\to X_0} f(X) \leq f(X_0);$$**

**(2) $f(X)$在$X_0$下半连续的充要条件是$f(X)$在$X_0$的邻域内有下界且
$$\varliminf_{\Omega\ni X\to X_0} f(X) \geq f(X_0).$$**


**思路：**
1. **必要性**（上半连续 ⇒ 上极限 ≤ f(x₀)）：
由$S_{X_0}$和上半连续定义，任意极限a小于等于$\inf S_{X_0}$，得上极限也符合

2. **充分性**（上极限 ≤ f(x₀) ⇒ 上半连续）：
反证，设不是上半连续，有$f(X) \geq f(X_0) + \varepsilon,$根据有上界由致密性定理得到收敛子列，有极限a大于$\inf S_{X_0}$，故$\varlimsup_{\Omega\ni X\to X_0} f(X) \geq a > f(X_0)$

3. **同理可得第二问**


**其他：**
**这里函数的上极限与下极限可以用下述定义：用$S_{X_0}$记集合
$$\{ a \in \mathbb{R} \mid \exists\ \Omega \ni X_k \neq X_0, \ \text{使}\ \lim_{k\to\infty} X_k = X_0\ \text{及}\ \lim_{k\to\infty} f(X_k) = a \},$$
则
$$\varlimsup_{\Omega\ni X\to X_0} f(X) = \sup S_{X_0},\quad \varliminf_{\Omega\ni X\to X_0} f(X) = \inf S_{X_0}.$$**

### 第11题

**条件：**
设 $f_m(X) \ (m=1,2,\dots)$ 是 $\mathbb{R}^n$ 中紧集 $\Omega$ 上的连续函数列，且对任意 $X\in\Omega$，有

$$\sup_{m} f_m(X) < +\infty$$

记 $f(X) = \sup_{m} f_m(X)$

**题目：证明 $f(X)$ 在 $\Omega$ 上达到最小值。**


**思路：**
1. **证 $f$ 下半连续**：
因为每个 $f_m$ 连续，用上确界的定义可以推出：$f(x_0)\le\liminf_{x\to x_0}f(x)$即 $f$ 下半连续。

2. **取极小化序列**
记 $m = \inf_\Omega f$，存在点列 $x_k\in\Omega$ 使得 $f(x_k)\to m$。

3. **用紧性取收敛子列**
$\Omega$ 紧，故 $x_k$ 有子列收敛到某点 $x^* \in \Omega$。再用下半连续性：$f(x^*) \le \liminf_{x_k} f(x_k) = m$，而 $f(x^*) \ge m$ 恒成立，故 $f(x^*) = m$，即最小值可达。
