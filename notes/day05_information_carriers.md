# Day 5 Notes: Information Carriers

## 阅读范围

本次阅读范围是论文 *Changing Base without Losing Space* 的 Section 3，重点理解作者如何从 SOLE encoding 中抽象出一般性的 information carrier 技术。

主要内容包括：

- SOLE 中 `spill` 的来源；
- information carrier 的基本模型；
- Lemma 4 的构造；
- 编码与解码过程；
- 两部分冗余的推导；
- information carrier 与后续向量表示、在线 prefix-free encoding 的关系。

这一部分承接 Day 4：SOLE 已经展示了局部换基的可行性，而 Section 3 将其中的核心机制抽象成可重复使用的引理。现有 Day 4 notes 也将 SOLE 定位为 information carrier lemma 的 warm-up。

------

## 内容总结

Information carrier 要解决的问题是：某个中间值可能来自大小不是 2 的幂的区间，因此不能恰好使用整数个 bits 存储。如果直接向上取整，就会产生空间浪费。论文将这种暂时无法直接写入二进制内存的中间值称为 `spill`。

作者的核心思路是，不单独保存 spill，而是使用另一个较大的值作为 information carrier。设旧 spill 为 $y$，carrier 为 $x$。算法把二者局部转换成可直接写入内存的 $m$ 和新的 spill $s$：
$$
(x,y)\longmapsto(m,s).
$$
其中 $m$ 来自大小恰好为 $2^M$ 的区间，因此可以使用恰好 $M$ bits 存储。旧 spill $y$ 能够仅从 $m$ 中恢复，而 carrier $x$ 可以从 $m$ 和新 spill $s$ 中恢复。

该构造最重要的性质是：它不会丢失信息，并且每次转换的冗余只有
$$
O\left(\frac{1}{\sqrt X}\right),
$$
其中 $X$ 是 carrier 的取值范围大小。当 $X$ 足够大时，每一步的空间损失就非常小。

------

## 论文具体解析

## 从 SOLE 到 Information Carrier

在 SOLE encoding 中，算法处理若干输入 blocks 后，会输出大部分信息，但仍留下一个来自非标准区间的中间值。例如，处理 $2n$ 个 blocks 后，可能留下一个来自
$$
[B+4n]
$$
的值。

论文将这个尚未写入标准二进制 blocks 的值称为 `spill`。

下一批输入数据不仅包含自身的信息，还可以帮助旧 spill 写入内存。因此，新输入数据起到了 information carrier 的作用。转换完成后：

```
旧 spill y + carrier x
        ↓
memory content m + 新 spill s
```

SOLE 使用特定的 short/long ranges 完成这一过程，而 information carrier lemma 给出了更一般的构造。

------

## Information Carrier 的基本模型

设：
$$
x\in[X],\qquad y\in[Y].
$$
其中：

- $x$ 是 information carrier；
- $y$ 是当前需要保存的旧 spill。

目标是构造单射：
$$
[X]\times[Y]
\longrightarrow
[2^M]\times[S],
$$
并记输出为：
$$
(x,y)\longmapsto(m,s).
$$
其中：

- $m\in[2^M]$ 是可以直接写入 $M$ bits 内存的值；
- $s\in[S]$ 是新的 spill。

该映射需要满足：

1. $y$ 可以仅从 $m$ 中恢复；
2. $x$ 可以从 $m$ 和 $s$ 中恢复；
3. 映射和逆映射都能在常数时间完成；
4. 产生的空间冗余很小。

旧 spill 必须只依赖 $m$ 才能恢复。否则，恢复 $y$ 还需要继续读取 $s$，而恢复 $s$ 又可能依赖下一个 spill，最终会形成很长的依赖链并破坏局部性。

------

## Lemma 4

论文的 information carrier lemma 可以概括为：

设 $X,Y\leq 2^w$。存在参数 $M$ 和 $S$，使得可以将
$$
(x,y)\in[X]\times[Y]
$$
单射映射为
$$
(m,s)\in[2^M]\times[S],
$$
并满足：
$$
S=O(\sqrt X),
$$
以及总冗余：
$$
M+\log_2 S-\log_2 X-\log_2 Y
=
O\left(\frac{1}{\sqrt X}\right).
$$
编码和解码只需要常数次整数乘法、除法和取模操作。

------

## 参数 $C$、$M$ 和 $S$

算法先选择 $M$，并定义：
$$
C=
\left\lfloor
\frac{2^M}{Y}
\right\rfloor.
$$
这意味着 $M$ 个 memory bits 除了能够保存 $y$，还可以额外保存一个来自 $[C]$ 的值。

接着，将 carrier $x$ 分解为商和余数：
$$
c=x\bmod C,
$$
其中：
$$
c\in[C],
$$
而新的 spill 范围大小为：
$$
S=
\left\lceil
\frac{X}{C}
\right\rceil.
$$
然后把 $y$ 和 $c$ 一起写入 memory：
$$
m=yC+c.
$$
由于：
$$
m<YC\leq 2^M,
$$
所以 $m$ 确实属于 $[2^M]$，可以使用恰好 $M$ bits 存储。

------

## 解码过程与正确性

从 $m$ 中可以恢复：
$$
y=
\left\lfloor
\frac{m}{C}
\right\rfloor,
$$
以及：
$$
c=m\bmod C.
$$
再结合新 spill $s$，可以恢复：
$$
x=sC+c.
$$
因此：

```
m        可以恢复 y；
m 和 s   可以恢复 x。
```

原来的 $(x,y)$ 能够被唯一恢复，所以该映射是 injective，不会丢失信息。

------

## 冗余分析

整个构造的冗余来自两个 rounding 操作。

### 第一部分冗余

将 $[Y]\times[C]$ 放入 $[2^M]$ 时，由于 $YC$ 未必恰好等于 $2^M$，产生：
$$
R_1
=
M-\log_2(YC).
$$
论文证明：
$$
R_1
=
O\left(\frac{1}{C}\right).
$$

### 第二部分冗余

将 $x\in[X]$ 分解为：
$$
(c,s)\in[C]\times[S]
$$
时，由于：
$$
S=
\left\lceil
\frac{X}{C}
\right\rceil,
$$
产生：
$$
R_2
=
\log_2 C+\log_2 S-\log_2 X.
$$
论文证明：
$$
R_2
=
O\left(\frac{C}{X}\right).
$$
因此总冗余为：
$$
R_1+R_2
=
O\left(
\frac{1}{C}+\frac{C}{X}
\right).
$$
为了平衡两部分冗余，令：
$$
\frac{1}{C}
\approx
\frac{C}{X},
$$
得到：
$$
C=\Theta(\sqrt X).
$$
代入后：
$$
R_1+R_2
=
O\left(\frac{1}{\sqrt X}\right).
$$
平方根参数并不是任意选择，而是两种 rounding loss 的平衡点。

------

## 时间复杂度

编码过程主要包括：

```
一次除法或商余分解；
一次乘法；
常数次加法。
```

解码过程同样只需要常数次除法、取模和乘法。

在论文使用的 Word RAM 模型中，这些 word-level arithmetic operations 都视为 $O(1)$ 时间。由于 $X$、$Y$、$C$ 等范围参数固定，还可以提前计算相关常量。

