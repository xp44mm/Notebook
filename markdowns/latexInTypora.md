## 4.2 公式排版基础

Add $a$ squared and $b$ squared to get $c$ squared. Or, using a more mathematical approach: $a^2 + b^2 = c^2$

Add $a$ squared and $b$ squared
to get $c$ squared
$$
\begin{equation}
a^2 + b^2 = c^2
\end{equation}
$$
Einstein says
$$
\begin{equation}
E = mc^2 \label{clever1}
\end{equation}
$$

This is a reference to.

It's wrong to say
$$
\begin{equation}
1 + 1 = 3 \tag{dumb}
\end{equation}
$$

or
$$
\begin{equation}
1 + 1 = 4 \notag
\end{equation}
$$
Again\ldots
$$
\begin{equation*}
a^2 + b^2 = c^2
\end{equation*}
$$
or you can type less for the same effect:
$$
 a^2 + b^2 = c^2
$$
or if you like the long one:
$$
\begin{displaymath}
a^2 + b^2 = c^2
\end{displaymath}
$$
In text: $\lim_{n \to \infty}\sum_{k=1}^n \frac{1}{k^2}= \frac{\pi^2}{6}$.
In display:
$$
\lim_{n \to \infty}\sum_{k=1}^n \frac{1}{k^2}= \frac{\pi^2}{6}
$$

### 4.2.1 数学模式和文本

$x^{2} \geq 0 \qquad\textbf{for all }x\in\mathbb{R}$

## 4.3 数学符号

### 4.3.1 一般符号

$a_1, a_2, \dots, a_n$

$a_1 + a_2 + \cdots + a_n$

### 4.3.2 指数、上下标和导数

$p^3_{ij} \qquad m_\mathrm{Knuth} $

$\qquad\sum_{k=1}^3 k$

$a^x+y \neq a^{x+y}\qquad e^{x^2} \neq {e^x}^2$ 

$f(x) = x^2 \quad f’(x)= 2x \quad f’’^{2}(x) = 4$

### 4.3.3 分式和根式

In display style:
$$
3/8 \qquad \frac{3}{8} \qquad \tfrac{3}{8}
$$
In text style:
$1\frac{1}{2}$ hours $1\dfrac{1}{2}$ hours

$\sqrt{x} \Leftrightarrow x^{1/2}\quad \sqrt[3]{2}\quad \sqrt{x^{2} + \sqrt{y}}$

Pascal's rule is
$$
\binom{n}{k} =\binom{n-1}{k}+ \binom{n-1}{k-1}
$$

### 4.3.4 关系符

$$
f_n(x) \stackrel{*}{\approx} 1
$$

### 4.3.5 算符

$$
\lim_{x \rightarrow 0}
\frac{\sin x}{x}=1
$$

$$
a\bmod b \\
x\equiv a \pmod{b}
$$

$$
\DeclareMathOperator{\argh}{argh}
\DeclareMathOperator*{\nut}{Nut}
\argh 3 = \nut_{x=1} 4x
$$

### 4.3.6 巨算符

行内：

$\sum_{i=1}^n \quad \int_0^{\frac{\pi}{2}} \quad \oint_0^{\frac{\pi}{2}} \quad \prod_\epsilon$

行间：
$$
\sum_{i=1}^n \quad
\int_0^{\frac{\pi}{2}} \quad
\oint_0^{\frac{\pi}{2}} \quad
\prod_\epsilon
$$
巨算符上下标位置：

行内：$\sum\limits_{i=1}^n \quad \int\limits_0^{\frac{\pi}{2}}  \quad\prod\limits_\epsilon$

行间：
$$
\sum\nolimits_{i=1}^n \quad \\
\int\limits_0^{\frac{\pi}{2}} \quad \\
\prod\nolimits_\epsilon
$$
多行上下标：
$$
\sum_{\substack{0\le i\le n \\
j\in \mathbb{R}}}
P(i,j) = Q(n)
$$
`subarray` 环境令多行表达式可选择居中 (c) 或左对齐 (l)：
$$
\sum_{\begin{subarray}{l}
0\le i\le n \\
j\in \mathbb{R}
\end{subarray}}
P(i,j) = Q(n)
$$

### 4.3.7 数学重音和上下括号

下例左边对“符号加下标”使用重音，右边对某个符号使用重音:

$\bar{x_0} \quad \bar{x}_0$
$\vec{x_0} \quad \vec{x}_0$
$\hat{\mathbf{e}_x} \quad\hat{\mathbf{e}}_x$

为多个字符加重音

$0.\overline{3} =\underline{\underline{1/3}}$
$\hat{XY} \qquad \widehat{XY}$
$\vec{AB} \qquad\overrightarrow{AB}$

`\overbrace` 和 `\underbrace` 命令用来生成上下括号，各自可带一个上下标公式。
$$
\underbrace{\overbrace{a+b+c}^6\cdot \overbrace{d+e+f}^7}_\text{meaning of life} = 42
$$

### 4.3.8 箭头

除了作为上下标之外，箭头还用于表示过程。amsmath 的 `\xleftarrow` 和 `\xrightarrow` 命令可以为箭头增加上下标：
$$
a\xleftarrow{x+y+z} b
$$

$$
c\xrightarrow[x<y]{a*b*c}d
$$

### 4.3.9 括号和定界符

${a,b,c} \neq \{a,b,c\}$

使用 `\left` 和 `\right` 命令可令括号（定界符）的大小可变，在行间公式中常用。
$$
1 + \left(\frac{1}{1-x^{2}}
\right)^3 \qquad
\left.\frac{\partial f}{\partial t}
\right|_{t=0}
$$
自定义大小的定界符：

$\Bigl((x+1)(x-1)\Bigr)^{2}$
$$
\bigl( \Bigl( \biggl( \Biggl(
\quad
\bigr\} \Bigr\} \biggr\} \Biggr\}
\quad
\big| \Big| \bigg| \Bigg|
\quad
\big\Downarrow \Big\Downarrow \bigg\Downarrow \Bigg\Downarrow
$$

## 4.4 多行公式

### 4.4.1 长公式折行

通常来讲应当避免写出需要折行的长公式。如果一定要折行的话，优先在等号之前折行，其次在加号、减号之前，再次在乘号、除号之前。其它位置应当避免折行。

`multline` 环境提供了书写折行长公式的方便环境。它允许用 `\\` 折行，将公式编号放在最后一行。多行公式的首行左对齐，末行右对齐，其余行居中。
$$
\begin{multline}
a + b + c + d + e + f + g + h + i \\
= j + k + l + m + n\\
= o + p + q + r + s\\
= t + u + v + x + z
\end{multline}
$$

### 4.4.2 多行公式

`align` 环境，它将公式用 `&` 隔为两部分并对齐。分隔符通常放在等号左边：
$$
\begin{align}
a & = b + c \\
& = d + e
\end{align}
$$
`align` 环境会给每行公式都编号。我们仍然可以用 `\notag` 去掉某行的编号。在以下的例子，为了对齐加号，我们将分隔符放在等号右边，这时需要给等号后添加一对括号 `{}` 以产生正常的间距：
$$
\begin{align}
a ={} & b + c \\
={} & d + e + f + g + h + i
+ j + k + l \notag \\
& + m + n + o \\
={} & p + q + r + s
\end{align}
$$
`align` 还能够对齐多组公式，除等号前的 `&` 之外，公式之间也用 `&` 分隔：
$$
\begin{align}
a &=1 & b &=2 & c &=3 \\
d &=-1 & e &=-2 & f &=-5
\end{align}
$$
如果我们不需要按等号对齐，只需罗列数个公式，`gather` 将是一个很好用的环境：
$$
\begin{gather}
a = b + c \tag{1}\\
d = e + f + g \tag{2}\\
h + i = j + k \notag \\
l + m = n \tag{3}
\end{gather}
$$

### 4.4.3 公用编号的多行公式

多个公式组在一起公用一个编号，编号位于公式的居中位置。
$$
\begin{equation}
\begin{aligned}
a &= b + c \\
d &= e + f + g \\
h + i &= j + k \\
l + m &= n
\end{aligned}
\end{equation}
$$

split 环境和 aligned 环境用法类似，也用于和 equation 环境套用，区别是 split 只能将每行的一个公式分两栏，aligned 允许每行多个公式多栏。

## 4.5 数组和矩阵

$$
\mathbf{X} = \left(
\begin{array}{cccc}
x_{11} & x_{12} & \ldots & x_{1n}\\
x_{21} & x_{22} & \ldots & x_{2n}\\
\vdots & \vdots & \ddots & \vdots\\
x_{n1} & x_{n2} & \ldots & x_{nn}\\
\end{array} \right)
$$

$$
|x| = \left\{
\begin{array}{rl}
-x & \text{if } x < 0,\\
0 & \text{if } x = 0,\\
x & \text{if } x > 0.
\end{array} \right.
$$

$$
|x| =
\begin{cases}
-x & \text{if } x < 0,\\
0 & \text{if } x = 0,\\
x & \text{if } x > 0.
\end{cases}
$$

$$
\begin{matrix}
1 & 2 \\ 3 & 4
\end{matrix} \qquad
\begin{bmatrix}
x_{11} & x_{12} & \ldots & x_{1n}\\
x_{21} & x_{22} & \ldots & x_{2n}\\
\vdots & \vdots & \ddots & \vdots\\
x_{n1} & x_{n2} & \ldots & x_{nn}\\
\end{bmatrix}
$$

$$
\mathbf{H}=
\begin{bmatrix}
\dfrac{\partial^2 f}{\partial x^2} &
\dfrac{\partial^2 f}
{\partial x \partial y} \\[8pt]
\dfrac{\partial^2 f}
{\partial x \partial y} &
\dfrac{\partial^2 f}{\partial y^2}
\end{bmatrix}
$$

## 4.6 公式中的间距

$$
\int_a^b f(x)\mathrm{d}x
\qquad
\int_a^b f(x)\,\mathrm{d}x
$$

$$
\newcommand\diff{\,\mathrm{d}}
\begin{gather*}
\int\int f(x)g(y)
\diff x \diff y \\
\int\!\!\!\int
f(x)g(y) \diff x \diff y \\
\iint f(x)g(y) \diff x \diff y \\
\iint\quad \iiint\quad \idotsint
\end{gather*}
$$

## 4.7 数学符号的字体控制

### 4.7.1 数学字母字体

$\mathcal{R} \quad \mathfrak{R}\quad \mathbb{R}$
$$
\mathcal{L}= -\frac{1}{4}F_{\mu\nu}F^{\mu\nu}
$$
$\mathfrak{su}(2)$ and $\mathfrak{so}(3)$ Lie algebra

### 4.7.2 数学符号的尺寸

$$
P = \frac
{\sum_{i=1}^n (x_i- x)(y_i- y)}
{\displaystyle \left[
\sum_{i=1}^n (x_i-x)^2
\sum_{i=1}^n (y_i-y)^2
\right]^{1/2} }
$$

### 4.7.3 加粗的数学符号

$$
$\mu, M \qquad
\boldsymbol{\mu}, \boldsymbol{M}$
$$

## 4.8 定理环境

$$
\newtheorem{mythm}{My Theorem}[section]
\begin{mythm}\label{thm:light}
The light speed in vaccum
is $299,792,458\,\mathrm{m/s}$.
\end{mythm}
\begin{mythm}[Energy]
The relationship of energy,
momentum and mass is
\[E^2 = m_0^2 c^4 + p^2 c^2\]
where $c$ is the light speed
described in theorem \ref{thm:light}.
\end{mythm}
$$

### 4.8.1 amsthm宏包

$$
\begin{law} \label{law:box}
Don't hide in the witness box
\end{law}
\begin{jury}[The Twelve]
It could be you! So beware and
see law~\ref{law:box}.\end{jury}
\begin{jury}
You will disregard the last
statement.\end{jury}
\begin{mar}No, No, No\end{mar}
\begin{mar}Denis!\end{mar}
$$

### 4.8.2 证明环境和证毕符号

$$
\begin{proof}
For simplicity, we use
\[
E=mc^2
\]
That's it.
\end{proof}
$$

$$
\begin{proof}
For simplicity, we use
\[
E=mc^2 \qedhere
\]
\end{proof}
$$

Assuming $\gamma= 1/\sqrt{1-v^2/c^2}$, then
$$
\begin{align*}
E &= \gamma m_0 c^2 \\
p &= \gamma m_0v \qedhere
\end{align*}
$$

## 4.9 符号表

### 4.9.1 LaTeX 普通符号

表 4.4: 文本/数学模式通用符号

|         |      |              |          |
| :-----: | :--: | :----------: | :------: |
|  $\{$   | $\}$ |     $\$$     |   $\%$   |
| $\dag$  | $\S$ | $\copyright$ | $\dots$  |
| $\ddag$ | $\P$ |  $\pounds$   | $x\ y$空格 |

表 4.5: 希腊字母

`\Alpha`, `\Beta` 等希腊字母符号不存在，因为它们和拉丁字母 A,B 等一模一样；小写字母里也不存在`\omicron`，直接用 o 代替。
$\alpha$、$\beta$、$\gamma$、$\delta$、$\epsilon$、$\varepsilon$、$\zeta$、$\eta$、$\Gamma$、$\Delta$、$\Theta$、$\theta$、$\vartheta$、$\iota$、$\kappa$、$\lambda$、$\mu$、$\nu$、$\xi$、$\Lambda$、$\Xi$、$\Pi$、$\pi$、$\varpi$、$\rho$、$\varrho$、$\sigma$、$\varsigma$、$\tau$、$\Sigma$、$\Upsilon$、$\Phi$、$\upsilon$、$\phi$、$\varphi$、$\chi$、$\psi$、$\omega$、$\Psi$、$\Omega$、$\varGamma$、$\varDelta$、$\varTheta$、$\varLambda$、$\varXi$、$\varPi$、$\varSigma$、$\varUpsilon$、$\varPhi$、$\varPsi$、$\varOmega$

表 4.6: 二元关系符

所有的二元关系符都可以加 `\not` 前缀得到相反意义的关系符，例如 `\not=` 就得到不等号（同 `\ne`）。

$\approx$、$\asymp$、$\bowtie$、$\cong$、$\dashv$、$\doteq$、$\equiv$、$\frown$、$\ge$、$\geq$、$\gg$、$\in$、$\Join$、$\le$、$\leq$、$\ll$、$\mid$、$\models$、$\ne$、$\neq$、$\ni$、$\notin$、$\owns$、$\parallel$、$\perp$、$\prec$、$\preceq$、$\propto$、$\sim$、$\simeq$、$\smile$、$\sqsubset$、$\sqsubseteq$、$\sqsupset$、$\sqsupseteq$、$\subset$、$\subseteq$、$\succ$、$\succeq$、$\supset$、$\supseteq$、$\vdash$

表 4.7: 二元运算符

$\amalg$、$\ast$、$\bigcirc$、$\bigtriangledown$、$\bigtriangleup$、$\bullet$、$\cap$、$\cdot$、$\circ$、$\cup$、$\dagger$、$\ddagger$、$\diamond$、$\div$、$\land$、$\lhd$、$\lor$、$\mp$、$\odot$、$\ominus$、$\oplus$、$\oslash$、$\otimes$、$\pm$、$\rhd$、$\setminus$、$\sqcap$、$\sqcup$、$\star$、$\times$、$\triangleleft$、$\triangleright$、$\unlhd$、$\unrhd$、$\uplus$、$\vee$、$\wedge$、$\wr$

表 4.8: 巨算符

$\bigcap$、$\bigcup$、$\bigodot$、$\bigoplus$、$\bigotimes$、$\bigsqcup$、$\biguplus$、$\bigvee$、$\bigwedge$、$\coprod$、$\idotsint$、$\iiiint$、$\iiint$、$\iint$、$\int$、$\oint$、$\prod$、$\sum$

表 4.9: 数学重音符号

$\acute{a}$、$\bar{a}$、$\breve{a}$、$\check{a}$、$\ddddot{a}$、$\dddot{a}$、$\ddot{a}$、$\dot{a}$、$\grave{a}$、$\hat{a}$、$\mathring{a}$、$\tilde{a}$、$\vec{a}$、$\widehat{AAA}$、$\wideparen{AAA}$、$\widetilde{AAA}$

表 4.10: 箭头

$\Downarrow$、$\downarrow$、$\gets$、$\hookleftarrow$、$\hookrightarrow$、$\iff$、$\leadsto$、$\leftarrow$、$\Leftarrow$、$\leftharpoondown$、$\leftharpoonup$、$\leftrightarrow$、$\Leftrightarrow$、$\Longleftarrow$、$\longleftarrow$、$\longleftrightarrow$、$\Longleftrightarrow$、$\longmapsto$、$\Longrightarrow$、$\longrightarrow$、$\mapsto$、$\nearrow$、$\nwarrow$、$\rightarrow$、$\Rightarrow$、$\rightharpoondown$、$\rightharpoonup$、$\rightleftharpoons$、$\searrow$、$\swarrow$、$\to$、$\uparrow$、$\Uparrow$、$\updownarrow$、$\Updownarrow$

表 4.11: 作为重音的箭头符号

$\overleftarrow{AB}$、$\overleftrightarrow{AB}$、$\overrightarrow{AB}$、$\underleftarrow{AB}$、$\underleftrightarrow{AB}$、$\underrightarrow{AB}$

表 4.12: 定界符

$\{$、$\|$、$\}$、$\backslash$、$\downarrow$、$\Downarrow$、$\langle$、$\lbrace$、$\lbrack$、$\lceil$、$\lfloor$、$\rangle$、$\rbrace$、$\rbrack$、$\rceil$、$\rfloor$、$\uparrow$、$\Uparrow$、$\Updownarrow$、$\updownarrow$、$\vert$、$\Vert$

表 4.13: 用于行间公式的大定界符

$\arrowvert$、$\Arrowvert$、$\bracevert$、$\lgroup$、$\lmoustache$、$\rgroup$、$\rmoustache$

表 4.14: 其他符号

$\aleph$、$\angle$、$\bot$、$\Box$、$\cdots$、$\clubsuit$、$\ddots$、$\Diamond$、$\diamondsuit$、$\dots$、$\ell$、$\emptyset$、$\exists$、$\flat$、$\forall$、$\hbar$、$\heartsuit$、$\Im$、$\imath$、$\infty$、$\jmath$、$\lnot$、$\mho$、$\nabla$、$\natural$、$\neg$、$\partial$、$\prime$、$\Re$、$\sharp$、$\spadesuit$、$\surd$、$\top$、$\triangle$、$\vdots$、$\wp$

### 4.9.2 AMS 符号

表 4.15: AMS 希腊字母和希伯来字母

$\beth$、$\daleth$、$\digamma$、$\gimel$、$\varkappa$

表 4.16: AMS 二元关系符

$\approxeq$、$\backepsilon$、$\backsim$、$\backsimeq$、$\because$、$\between$、$\blacktriangleleft$、$\blacktriangleright$、$\bumpeq$、$\Bumpeq$、$\circeq$、$\curlyeqprec$、$\curlyeqsucc$、$\doteqdot$、$\eqcirc$、$\eqslantgtr$、$\eqslantless$、$\fallingdotseq$、$\geqq$、$\geqslant$、$\ggg$、$\gtrapprox$、$\gtrdot$、$\gtreqless$、$\gtreqqless$、$\gtrless$、$\gtrsim$、$\leqq$、$\leqslant$、$\lessapprox$、$\lessdot$、$\lesseqgtr$、$\lesseqqgtr$、$\lessgtr$、$\lesssim$、$\lll$、$\llless$、$\pitchfork$、$\precapprox$、$\preccurlyeq$、$\precsim$、$\risingdotseq$、$\shortmid$、$\shortparallel$、$\smallfrown$、$\smallsmile$、$\sqsubset$、$\sqsupset$、$\Subset$、$\subseteqq$、$\succapprox$、$\succcurlyeq$、$\succsim$、$\Supset$、$\supseteqq$、$\therefore$、$\thickapprox$、$\thicksim$、$\trianglelefteq$、$\triangleq$、$\trianglerighteq$、$\varpropto$、$\vartriangleleft$、$\vartriangleright$、$\vDash$、$\Vdash$、$\Vvdash$

表 4.17: AMS 二元运算符

$\barwedge$、$\boxdot$、$\boxminus$、$\boxplus$、$\boxtimes$、$\centerdot$、$\circledast$、$\circledcirc$、$\circleddash$、$\curlyvee$、$\curlywedge$、$\divideontimes$、$\dotplus$、$\doublebarwedge$、$\doublecap$、$\doublecup$、$\intercal$、$\leftthreetimes$、$\ltimes$、$\rightthreetimes$、$\rtimes$、$\smallsetminus$、$\veebar$

表 4.18: AMS 箭头

$\circlearrowleft$、$\circlearrowright$、$\curvearrowleft$、$\curvearrowright$、$\dashleftarrow$、$\dashrightarrow$、$\downdownarrows$、$\downharpoonright$、$\leftarrowtail$、$\leftleftarrows$、$\leftrightarrows$、$\leftrightharpoons$、$\leftrightsquigarrow$、$\Lleftarrow$、$\looparrowleft$、$\looparrowright$、$\Lsh$、$\multimap$、$\rightarrowtail$、$\rightleftarrows$、$\rightleftharpoons$、$\rightrightarrows$、$\rightsquigarrow$、$\Rrightarrow$、$\Rsh$、$\twoheadleftarrow$、$\twoheadrightarrow$、$\upharpoonleft$、$\upharpoonright$、$\upuparrows$

表 4.19: AMS 反义二元关系符和箭头

$\gnapprox$、$\gneq$、$\gneqq$、$\gnsim$、$\gvertneqq$、$\lnapprox$、$\lneq$、$\lneqq$、$\lnsim$、$\lvertneqq$、$\ncong$、$\ngeq$、$\ngeqq$、$\ngeqslant$、$\ngtr$、$\nleftarrow$、$\nLeftarrow$、$\nleftrightarrow$、$\nLeftrightarrow$、$\nleq$、$\nleqq$、$\nleqslant$、$\nless$、$\nmid$、$\nparallel$、$\nprec$、$\npreceq$、$\nrightarrow$、$\nRightarrow$、$\nshortmid$、$\nshortparallel$、$\nsim$、$\nsubseteq$、$\nsubseteqq$、$\nsucc$、$\nsucceq$、$\nsupseteq$、$\nsupseteqq$、$\ntriangleleft$、$\ntrianglelefteq$、$\ntriangleright$、$\ntrianglerighteq$、$\nvdash$、$\nvDash$、$\nVdash$、$\nVDash$、$\precnapprox$、$\precneqq$、$\precnsim$、$\subsetneq$、$\subsetneqq$、$\succnapprox$、$\succneqq$、$\succnsim$、$\supsetneq$、$\supsetneqq$、$\varsubsetneq$、$\varsubsetneqq$、$\varsupsetneq$、$\varsupsetneqq$

表 4.20: AMS 定界符

$\llcorner$、$\lrcorner$、$\lvert$、$\lVert$、$\rvert$、$\rVert$、$\ulcorner$、$\urcorner$

表 4.21: AMS 其它符号

$\angle$、$\backprime$、$\Bbbk$、$\bigstar$、$\blacklozenge$、$\blacksquare$、$\blacktriangle$、$\blacktriangledown$、$\circledS$、$\complement$、$\diagdown$、$\diagup$、$\eth$、$\Finv$、$\Game$、$\hbar$、$\hslash$、$\lozenge$、$\measuredangle$、$\mho$、$\nexists$、$\sphericalangle$、$\square$、$\triangledown$、$\varnothing$、$\vartriangle$


