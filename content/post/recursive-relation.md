---
title: "使用矩陣計算遞歸關係式"
type: post
date: 2019-04-19T22:26:37+08:00
---

最近與朋友討論一條題目：一個 $3 \times n$ 的網格，以 $1 \times 2$ 的磚塊（可旋轉）填滿，有多少種填法？

朋友的解法是動態規劃，我對動態規劃不太熟悉，倒是比較常用遞歸關係，兩者算是殊途同歸。

# 遞歸關係式

假設填法數量等於 $f(n)$。考慮 $3 \times n$ 網格的最頂層 (Row 1)，有以下幾種填法：

### 情況一：三個直排

    Row 1    ｜｜｜
    Row 2    ｜｜｜

這種情況下，下面的 $n - 2$ 行可以任意填，於是填法數量等於 $f(n - 2)$。

### 情況二：一個橫排、一個直排

這裡有兩種填法：

    Row 1    ————｜
    Row 2        ｜

以及

    Row 1    ｜————
    Row 2    ｜

由於兩種填法是對稱的，因此只考慮其中一種，然後把數量乘以二。

下面我們繼續考慮第二行怎麼填：

### 情況二之一：用橫排填

    Row 1    ————｜
    Row 2    ————｜

這種情況下，填法數量等於 $f(n - 2)$。

### 情況二之二：用直排填

    Row 1    ————｜
    Row 2    ｜  ｜
    Row 3    ｜    

這種情況下，填了一個直排後，剩下的空間只能再填直排：

    Row 1    ————｜
    Row 2    ｜｜｜
    Row 3    ｜｜  

於是第二行填滿了，第三行又只剩下一格，也是只能填直排：

    Row 1    ————｜
    Row 2    ｜｜｜
    Row 3    ｜｜｜
    Row 4        ｜

有趣的事情發生了：我們連續填到三個直排，結果情況回歸到一開始我們要選擇怎樣填第二行的模式。於是我們又有兩種填法：

    Row 1    ————｜
    Row 2    ｜｜｜
    Row 3    ｜｜｜
    Row 4    ————｜

以上對應 $f(n - 4)$；

    Row 1    ————｜
    Row 2    ｜｜｜
    Row 3    ｜｜｜
    Row 4    ｜｜｜
    Row 5    ｜｜｜
    Row 6        ｜

以上又繼續遞歸，直到底部。

因此，整個情況二的填法數目等於 $2(f(n - 2) + f(n - 4) + ... + f(0))$。

因此我們就有了 $f(n)$ 的遞歸關係式：

<p>$
f(0) = 1 \\
f(n) = f(n - 2) + 2(f(0) + f(2) + ... + f(n - 2))
$</p>

由於只有對偶數有定義，替換一下 $h(n) = f(2n)$ 讓它好看一點：

<p>$
h(0) = 1 \\
h(n) = h(n - 1) + 2(h(0) + h(1) + ... + h(n - 1))
$</p>

這個遞歸關係已經足夠寫出 $O(n)$ 的解法了：

```Python
def f(n):
    return h(n / 2)

def h(n):
    curr = 1 # h(0)
    prevSum = curr # sum of h(k) for k in 0..0, which is still h(0)
    for k in range(1, n + 1):
        # At this line, `curr` = h(k - 1), `prevSum` = h(0) + ... + h(k - 1)
        # We want at the end of this block, `curr` = h(k), `prevSum` = h(0) + ... + h(k)
        curr += 2 * prevSum
        prevSum += curr
    return curr
```

# 難度升級

然而原題目的重點在於 $n$ 的取值範圍：$n \le 10^{100}$，求填法數目 $\mod 1000000007$

這時 $O(n)$ 的算法明顯不夠用了，$O(\sqrt{n})$ 之類也不行，必須找到 $O(\log(n))$ 以內的算法。

朋友的解法是求動態規劃（Dynamic Programming）轉移方程，然後編碼成 $4 \times 4$ 矩陣，然後用快速冪 (fast exponentiation) 求解，複雜度 $O(\log(n))$。

我覺得這個遞歸關係可以直接解出解析式，但當場沒做出來，有點不甘心，回家後又算了算，發現其實不太難，跟 Fibonacci 差不多。

# 簡化遞歸關係

首先，注意到

<p>$
h(n) - h(n - 1) \\
= (h(n - 1) + 2(h(0) + h(1) + ... + h(n - 1))) - (h(n - 2) + 2(h(0) + h(1) + ... + h(n - 2))) \\
=(h(n - 1) - h(n - 2)) + 2 (h(0) - h(0) + h(1) - h(1) + ... + h(n - 2) - h(n - 2) + h(n - 1)) \\
= h(n - 1) - h(n - 2) + 2 h(n - 1) \\
= 3h(n - 1) - h(n - 2) \\
\therefore h(n) = 4h(n - 1) - h(n - 2)
$</p>

這個遞歸關係式和 Fibonacci 有着相似的形式，可以寫成矩陣形式計算：

<p>$
\begin{pmatrix}
h(n+1) \\
h(n)
\end{pmatrix}
 = 
\begin{pmatrix}
4 & -1 \\
1 & 0
\end{pmatrix}
\begin{pmatrix}
h(n) \\
h(n-1)
\end{pmatrix} \\
= \begin{pmatrix}
4 & -1 \\
1 & 0
\end{pmatrix}^{2}
\begin{pmatrix}
h(n-1) \\
h(n-2)
\end{pmatrix} \\
= \dots \\
= \begin{pmatrix}
4 & -1 \\
1 & 0
\end{pmatrix}^{n}
\begin{pmatrix}
h(1) \\
h(0)
\end{pmatrix} \\
= \begin{pmatrix}
4 & -1 \\
1 & 0
\end{pmatrix}^{n}
\begin{pmatrix}
3 \\
1
\end{pmatrix}
$</p>

這樣，只需要用快速冪求 `$
\begin{pmatrix}
4 & -1 \\
1 & 0
\end{pmatrix}^{n}
$`，便能以 $O(\log(n))$ 複雜度求出 $h(n)$ 和 $f(n)$。

```Python
def h(n):
    return h(n / 2)

def multiply2x2Matrix(m1, m2):
    return [
        (m1[0] * m2[0] + m1[1] * m2[2]) % 1000000007, (m1[0] * m2[1] + m1[1] * m2[3]) % 1000000007,
        (m1[2] * m2[0] + m1[3] * m2[2]) % 1000000007, (m1[2] * m2[1] + m1[3] * m2[3]) % 1000000007,
    ]

def h(n):
    a = [4, -1, 1, 0] # The squaring matrix
    m = [1, 0, 0, 1] # The result matrix
    while n > 0:
        if n % 2 == 1:
            m = multiply2x2Matrix(a, m)
        a = multiply2x2Matrix(a, a)
        n >>= 1
    # extract the result of the second row of m * vector(3, 1)
    return (m[2] * 3 + m[3]) % 1000000007
```

