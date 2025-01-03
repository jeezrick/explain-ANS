# ANS

---

[中文](Readme.md) | [English](Readme_en.md)

# UNDERSTANDING ANS

---

想象在某个平行宇宙中，我们是一种使用二进制的生物，我们会使用这样一串数字

```python
bin_num = b'010001001001'
```

因为我们是二进制生物，所以我们每次写数字都写的很长，消耗很多纸张，导致了热带雨林被砍伐，全球变暖，非常不环保。

后来一个关心环境的人想出来一种叫做“编码”的方法，可以将很长的数字融合成一个短的但是没有一个二进制人可以理解的数，他把他的方法中的这个数字叫做$state$。

```python
# 他的方法如下
def encode_trivial(state, symbol, mod):
    assert(symbol >=0 and symbol < mod)
    return state * mod + symbol

def decode_trivial(state, mod):
    return state // mod, state % mod
```

```python
# 使用他的编码方法
state = 0
for i in range(len(bin_num)):
    symbol = bin_num[-1 - i] - b'0'[0]
    print(f"{i}时刻编码的符号symbol_{i}: {symbol}, 当前的state_{i}: {state}")
    state = encode_trivial(state, symbol, 2)
print("最终state: ", state)
```

    0时刻编码的符号symbol_0: 1, 当前的state_0: 0
    1时刻编码的符号symbol_1: 0, 当前的state_1: 1
    2时刻编码的符号symbol_2: 0, 当前的state_2: 2
    3时刻编码的符号symbol_3: 1, 当前的state_3: 4
    4时刻编码的符号symbol_4: 0, 当前的state_4: 9
    5时刻编码的符号symbol_5: 0, 当前的state_5: 18
    6时刻编码的符号symbol_6: 1, 当前的state_6: 36
    7时刻编码的符号symbol_7: 0, 当前的state_7: 73
    8时刻编码的符号symbol_8: 0, 当前的state_8: 146
    9时刻编码的符号symbol_9: 0, 当前的state_9: 292
    10时刻编码的符号symbol_10: 1, 当前的state_10: 584
    11时刻编码的符号symbol_11: 0, 当前的state_11: 1169
    最终state:  2338

可以看见他把需要编码的数叫做$symbol$，比如上面的例子中，一开始的$state$为 0，编码了$symbol$ "1" 之后，$state$就变成了 1，后面又变成了 2，4，9，18，36...

```python
# 使用他的解码方法
print("初始state: "state)
while state > 0:
    state, val = decode_trivial(state, 2)
    print(state, val)
```

    2338
    1169 0
    584 1
    292 0
    146 0
    73 0
    36 1
    18 0
    9 0
    4 1
    2 0
    1 0
    0 1

---

在另一个时空维度，我们是十进制生物，我们不仅把十进制的数编码成二进制让更笨的被关在计算机里的二进制生物理解，我们还非常贪心的想使用更神奇的超越进制的方法把十进制编码的更短，涉足神的领域

假设$s$当前需要编码的符号，即 symbol。

s 的数量为 n，它们符合以下关系

$$ s \in A = [s_1, s_2, s_3, ... s_{n- 1}] \tag{1} $$

$$ \{p*s\}, s \in A, \sum*{i = 0}^{N}{p\_{s_i}} = 1 \tag{1}$$

通常使用整数的统计频率来代表概率，本文使用这个例子

$$ s \in A = [a, b, c, d, e], \text{in which $s_1 = a, s_2 = b,...,s_{n - 1} = A[n - 1]$} \tag{1}$$

$$F_s \in FreqTable_s = [2, 5, 8, 0, 1]\tag{1}$$

$F_{s}$为$s$的统计频率，$M$为所有 symbol 的频率之和。

$$M = \sum_{i = 0}^{n}{F_{s_i}} = 2 + 5 + 8 + 0 + 1 = 16\tag{1}$$

```python
s_table = ["a", "b", "c", "d", "e"]
freq_table = [2, 5, 8, 0, 1]
M = sum(freq_table)
M
```

    16

ANS 的编解码公式

编码公式

$C: x_{t + 1} = C(x_t, s_t)$
$$ X*t = \lfloor \frac{X*{t - 1}}{F*{s_t}} \rfloor \* M + Cumu*{s*t} + mod(X*{t - 1}, F\_{s_t}) \tag{1}$$

其中，$Cumu_s$为$s$的累计频率，满足

$$
Cumu_{s_{A[n]}} =
  \begin{cases}
    F_{s_{n}} + Cumu_{s_{n - 1}}, &\text{if $n > \mathcal{0}$}\\
	0, &\text{if $n == \mathcal{0}$,}
  \end{cases} \tag{1}
$$

则
$$FreqTable_s = [2, 5, 8, 0, 1]\tag{1}$$
$$ Cumu_s \in CumuTalbe_s = [0, 2, 7, 15, 15, 16] \tag{1}$$

```python
cumu_table = [0, 2, 7, 15, 15, 16]
```

则编码函数可以表示为

```python
def C(X_t, s_t):
    cumu = cumu_table[s_t]
    freq = cumu_table[s_t + 1] - cumu_table[s_t]
    return (X_t // freq * M + cumu + X_t % freq)
```

```python
X = 0
text = "ebacabecc"
print(f"state0 is {X}")
for i in range(len(text)):
    symbol_index = s_table.index(text[i])
    X = C(X, symbol_index)
    print(f"symbol{i + 1} is {text[i]}, state{i + 1} is {X}")
import math
print(math.log2(X))
```

    state0 is 0
    symbol1 is e, state1 is 15
    symbol2 is b, state2 is 50
    symbol3 is a, state3 is 400
    symbol4 is c, state4 is 807
    symbol5 is a, state5 is 6449
    symbol6 is b, state6 is 20630
    symbol7 is e, state7 is 330095
    symbol8 is c, state8 is 660190
    symbol9 is c, state9 is 1320381
    20.332522853073755

解码公式

$D: s_t, x_t = D(x_{t + 1})$
$$ slot = mod(X*{t+1}, M) \tag{1}$$
$$ s_t = Cumu_inv(slot) \tag{1}$$
$$ X*{t} = \lfloor \frac{X*{t+1}}{M} \rfloor \* F*{s*{t}} + slot - Cumu*{s_t} \tag{1}$$

ANS 通过一种类似栈的方式将$s$编码到一个总的数字当中,即 last in first out，这个数字被称为 state，设为$X$，则 t 时刻的 state 表示为$X_t$，t 时刻的 symbol 表示为$s_t$。

ANS 的编解码公式

```python
def BinarySearch(slot):
    low = 0
    high = len(cumu_table) - 1
    mid = low + (high - low) // 2
    while(high - low > 1):
        mid = low + (high - low) // 2
        if (slot >= cumu_table[mid]):
            low = mid
        else:
            high = mid
    return low

def D(X_t):
    slot = X_t % M
    print(f"slot is {slot}, ", end="")
    s_t = BinarySearch(slot)
    freq = cumu_table[s_t + 1] - cumu_table[s_t]
    cumu = cumu_table[s_t]
    return s_t, (X_t // M * freq + slot - cumu)
```

```python
X_decode = X
print("symbol table is ", s_table)
print("freq table is ", freq_table)
print("cumu table is ", cumu_table)
state_index = 9
print(f"state{state_index} is {X_decode}")
while (X_decode):
    symbol_index, X_decode = D(X_decode)
    print(f"symbol{state_index} is {s_table[symbol_index]}, state{state_index - 1} is {X_decode}")
    state_index -= 1
```

    symbol table is  ['a', 'b', 'c', 'd', 'e']
    freq table is  [2, 5, 8, 0, 1]
    cumu table is  [0, 2, 7, 15, 15, 16]
    state9 is 1320381
    slot is 13, symbol9 is c, state8 is 660190
    slot is 14, symbol8 is c, state7 is 330095
    slot is 15, symbol7 is e, state6 is 20630
    slot is 6, symbol6 is b, state5 is 6449
    slot is 1, symbol5 is a, state4 is 807
    slot is 7, symbol4 is c, state3 is 400
    slot is 0, symbol3 is a, state2 is 50
    slot is 2, symbol2 is b, state1 is 15
    slot is 15, symbol1 is e, state0 is 0

下面分析以下编解码过程的结果：
ENCODING:
state0 is 0
symbol1 is e, state1 is 15
symbol2 is b, state2 is 50
symbol3 is a, state3 is 400
symbol4 is c, state4 is 807
symbol5 is a, state5 is 6449
symbol6 is b, state6 is 20630
symbol7 is e, state7 is 330095
symbol8 is c, state8 is 660190
symbol9 is c, state9 is 1320381
20.332522853073755

symbol table is ['a', 'b', 'c', 'd', 'e']
freq table is [2, 5, 8, 0, 1]
cumu table is [0, 2, 7, 15,15, 16]

DECODING:
state9 is 1320381
slot is 13, symbol9 is c, state8 is 660190
slot is 14, symbol8 is c, state7 is 330095
slot is 15, symbol7 is e, state6 is 20630
slot is 6, symbol6 is b, state5 is 6449
slot is 1, symbol5 is a, state4 is 807
slot is 7, symbol4 is c, state3 is 400
slot is 0, symbol3 is a, state2 is 50
slot is 2, symbol2 is b, state1 is 15
slot is 15, symbol1 is e, state0 is 0
从上面的编解码结果中可以分析出很多有趣的事。

首先我们可以看见编码的符号被倒序解码出来。这是 ANS 的特性，类似于栈，last in first out。用更形象的话来说，编码就像不断地在巧克力球外面包一层锡纸，那么解码的时候，最后被包上的那一层是最先被解开的，最先被包上的那一层是最后解开的。同时我们也可以看见编解码的 state 是一一对应的，这也和包锡纸的比喻贴合。第三，注意看 state8 和 state7 之间的差值大小，可以发现，state8 = state7\*2，也就是说计算机表示 state8 比表示 state7 多用了一个 bit。而从 state7 到 state8 编码的符号 symbol8 是 c，c 的概率是$ \frac{8}{16}$，熵为$\log\_{2}\frac{16}{8} = 1$，也就是说，编码符号 c 所使用的 bit 数，正好是它的理论熵，这是熵编码接近理论熵的一个表现。下面分析这些现象是怎么来的。

首先

$C: x_{t + 1} = C(x_t, s_t)$
$$ X*t = \lfloor \frac{X*{t - 1}}{F*{s_t}} \rfloor \* M + Cumu*{s*t} + mod(X*{t - 1}, F\_{s_t}) \tag{1}$$

那么

$$H_{s_t} = \log_{2}\frac{M}{F_{s_t}}$$
$$\frac{X_t}{X_{t - 1}} \approx \frac{M}{F_{s_t}}$$
$$\log_{2}{X_t} = \log_{2}X_{t- 1}\frac{M}{F_{s_t}} = \log_{2}X_{t - 1} + \log_{2}\frac{M}{F_{s_t}} = \log_{2}X_{t - 1} + H_{s_t}$$

这也就意味着，每编码一个符号，ANS 的 state 增加的 bit 数就是这个符号的熵。当然实际上是非常接近，注意上面$(1)$中的约等于。

熵编码是如何解码时找到符号后又完美的返回上一个 state 的呢？

根据解码公式：

$$ slot = mod(X*t, M) $$
$$= (\lfloor \frac{X*{t - 1}}{F*{s_t}} \rfloor \* M + Cumu*{s*t} + mod(X*{t - 1}, F*{s_t})) \bmod M $$
$$= Cumu*{s*t} + mod(X*{t - 1}, F\_{s_t})$$

显然$slot$是处于$CumuTalbe_s$中专属于$s_t$的那一段空间的，所以
$$ CumuTalbe*s[s*{n}]< slot < CumuTalbe*s[s*{n + 1}] \tag{1}$$

也就是说，$CumuTalbe_s$所表示的数轴区间被分成了不同长度的子区间，而每一个子区间都唯一对应一个符号，只要使得解码时获取到这个符号对应的子区间，就可以知道这个区间对应的符号。

看到这里的读者心里可能会疑惑，这里需要这么麻烦吗？单纯的顺序给每个符号一个 id 不就行了吗，比如['a', 'b', 'c', 'd', 'e']对应[1, 2, 3, 4, 5]，这样不是更简单吗？

这一点就和 ANS 的另一个设计有关。ANS 是怎么完整地回到上一状态的。

解码公式中

$$ X*{t - 1} = \lfloor \frac{X*{t}}{M} \rfloor \* F*{s*{t}} + slot - Cumu\_{s_t} \tag{1}$$

我们知道在编码公式中

$$ X*t = \lfloor \frac{X*{t - 1}}{F*{s_t}} \rfloor \* M + Cumu*{s*t} + mod(X*{t - 1}, F\_{s_t}) \tag{1}$$

注意

$$ Cumu*{s_t} + mod(X*{t - 1}, F\_{s_t}) < M \tag{1}$$

则将（1）带入 2 中可得

$$ X*{t - 1} =\lfloor \frac{X*{t - 1}}{F*{s_t}} \rfloor \* F*{s*{t}} + slot - Cumu*{s_t} \tag{1}$$

而现在这个公式就很明显的展示出来，解码公式中的

$$slot - Cumu_{s_t} = X_{t - 1} - \lfloor \frac{X_{t - 1}}{F_{s_t}} \rfloor * F_{s_{t}} = \text{roundoff error} \tag{1}$$

即$slot - Cumu_{s_t}$是用来补足$\lfloor \frac{X_{t - 1}}{F_{s_t}} \rfloor$ 的舍入误差的，而
$$slot - Cumu_{s_t} = Cumu_{s_t} + mod(X_{t - 1}, F_{s_t}) - Cumu_{s_t} = mod(X_{t - 1}, F_{s_t})$$

$mod(X_{t - 1}, F_{s_t})$就正是$\lfloor \frac{X_{t - 1}}{F_{s_t}} \rfloor$时向下取整丢掉的值。

比如$state_8 = 660190$，要编码的符号为$c$，频率为$8$，$M= 16$。

则$660190 / 8 = 82523 , 6$。

而$state_9 = 1320381$，$state_9 /M * F_{st}= 660184$, 正好差$state_8 % F_{st} = 6$，而这个$6$我们通过$slot - cumust$获得。

那么现在可以明了的说，在编码公式中，第一项是为了模拟熵，第二项和第三项是为了在解码时可以找回符号的同时，补足第一项的向下取整误差使得 s 完整地返回上一个 state。这也就是说，如果可以想出别的满足这些条件的构造方式，公式完全可以修改。这也引出了我们优化的点。
