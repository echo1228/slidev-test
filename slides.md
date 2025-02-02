---
theme: prussianblue
contents: 
  - 算法引入
  - 算法概述
  - 代码解析
  - 加强练习
---

# 树状数组

---
id: 1
---

# 数据结构与算法
树状数组会是我们学到的第一个高级数据结构，学完树状数组，我们便可以处理一部分带修改的区间查询问题。

在计算机的领域，我们通常会将数据结构与算法分开，但是又把他们放在一起学。

因为本质上来说数据结构与算法都是用代码解决一些问题，区别在于数据结构是一力降十会，而算法是四两拨千斤。

所以大部分时候，数据结构都有着冗长的代码。

---
id: 1
---

# 引入问题
<div class="rounded-md shadow-md border-2 h-50 mx-auto p-4 m-4">

给出一个长度为 $n$ 的数组 $a$，需要进行 $q$ 次操作。($n,q \le 2\times 10^5$)

要求支持以下两种操作:

- 查询区间 $[l, r]$ 的和。
- 令 $a[i] = a[i] + x$。

</div>

<div class="rounded-md shadow-md border-2 h-35 p-4 m-4 float-left">

<div class="i-ph-bell-ringing bg-red-800"></div> 

回想一下如果只需要支持第一种操作，可以用什么算法？

为什么不能在那个算法的基础上进行第二种操作？

</div>


---
id: 1
---

# 前缀和

我们先来回顾一下这个简单而又强大的算法**前缀和**。

前缀和算法是预处理出所有**前缀**的和，任意一段区间的和就可以被表示成两段前缀和相减。

（扩展：任何有逆运算的信息都可以用类前缀和算法来查询区间信息）

那么我们来考虑一下修改，如果我们要修改第 $i$ 个数字，那么相对应的就要修改区间 $\{i,i+1\dots n\}$ 的所有前缀和数组的值。

```cpp
using ll = long long;
const int N = 2e5+5;
ll n, a[N], q[N];
void modify(int i, int x) {
  for(int j = i; j <= n; j ++) {
    q[j] = q[j] + x;
  }
}
```

单次修改的时间复杂度退化至 $O(n)$

---
id: 1
---

<img src="/luxun.jpg" class="h-80 float-left m-4"/>

鲁迅曾经说过：“中国人的性情是总喜欢调和折中的，譬如你说，这屋子太暗，须在这里开一个窗，大家一定是不允许的。但是如果你主张拆掉屋顶，他们就来调和，愿意开窗了”。

所以，前缀和算法，单次查询时间复杂度为 $O(1)$, 单次修改时间复杂度为 $O(n)$，评测机一定是不允许的。

但是如果有一个神奇的算法可以调和折中一下，可以做到单次查询和修改都是 $O(\log{n})$ 或者 $O(\sqrt{n})$，评测机就会欣然接受了。 

那我们想想，如果我们可以把一段区间分成 $O(\log{n})$ 或者 $O(\sqrt{n})$ 块相加，问题就迎刃而解了。

---
id: 1
---

# 分块算法雏形

今日我们不学习分块，但是我们可以先做了解。

把一个数组分为 $\sqrt{n}$ 块，块内信息整体保存，若查询时遇到两边不完整的块直接暴力查询。

因为块长等于 $\frac{n}{\sqrt{n}}=\sqrt{n}$ 暴力查询次数不会超过 $2\times \sqrt{n}$。

同时总共也只有 $\sqrt{n}$ 块，查询块整体信息也不会超过 $\sqrt{n}$ 次。

修改同理，整体时间复杂度可以做到单次查询或修改 $O(\sqrt{n})$。

$\begin{matrix} \sqrt{n} 块 \\ \overbrace{ \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad\qquad \qquad \qquad \qquad \quad} \end{matrix}$

<div class="bg-gray-300 h-10 w-30 rounded-lg flex justify-center float-left m-1">

$len \leq \sqrt{n}$
</div>
<div class="bg-gray-300 h-10 w-30 rounded-lg flex justify-center float-left m-1">

$len \leq \sqrt{n}$
</div>
<div class="bg-gray-300 h-10 w-30 rounded-lg flex justify-center float-left m-1">

$len \leq \sqrt{n}$
</div>
<div class="bg-gray-300 h-10 w-50 rounded-lg flex justify-center float-left m-1">

$\cdots$
</div>
<div class="bg-gray-300 h-10 w-30 rounded-lg flex justify-center float-left m-1">

$len \leq \sqrt{n}$
</div>

<br>
<br>

分块算法的优点是代码简短，能解决的问题模型多。

缺点是单次查询的时间复杂度仍旧不算特别优秀。

---
id: 2
---

# 树状数组-初步了解

给出一个长度为 $n$ 的数组 $a$，现在我想知道 $a[1\cdots 7]$ 的和。

在没有其他信息的情况下，你一定会告诉我 $a[1\cdots 7] = a[1]+a[2]+a[3]+\cdots+a[7]$。

但是如果我告诉你一些信息 $A = a[1]+a[2]+a[3]+a[4],B=a[5]+a[6],C=a[7]$。

如果你是个聪明的孩子，你一定会不假思索的告诉我，$a[1\cdots 7]=A+B+C$。

树状数组就是把每段前缀都分成不超过 $\log{n}$ 段。这样我就可以在 $\log{n}$ 的时间复杂度内计算出仍以一段区间的信息。

我们定义数组 $tr$，$tr[i]$ 表示区间 $[i-lowbit(i)+1, i]$ 的和。

这样，每段前缀就会被分成不超过 $\log{n}$ 段的数据

---
id: 2
---

# 树状数组-lowbit

那什么是lowbit？

我们定义 lowbit(n) 表示非负整数 n 在二进制下“最低位的1及其更低位的0”组成的新的二进制数。

上面那句话有点绕，我们更形式化的来描述一下。

如果非负整数 $n$ 的二进制中最低位 $1$ 所在的位置是 $i$，$lowbit(n)=2^i$。

<div class="overflow-auto h-50 w-180 mx-auto">

| 非负整数 | 二进制形式  | 最低位1的位置 | lowbit |
|:----:|:------:|:-------:|:-------:|
| 1    | 000001 | 0       | $2^0=1$ |
| 2    | 000010 | 1       | $2^1=2$ |
| 3    | 000011 | 0       | $2^0=1$ |
| 4    | 000100 | 2       | $2^2=4$ |
| 5    | 000101 | 0       | $2^0=1$ |
| 6    | 000110 | 1       | $2^1=2$ |
| 7    | 000111 | 0       | $2^0=1$ |
| 8    | 001000 | 3       | $2^3=8$ |

</div>

---
id: 2
---

# 树状数组-lowbit练习
基于以下两点，完成练习。
- $tr[i]$ 表示区间 $[i-lowbit(i)+1, i]$ 的和
- 非负整数 $n$ 的二进制中最低位 $1$ 所在的位置是 $i$，$lowbit(n)=2^i$

<div class="rounded-md shadow-md border-2 h-70 mx-auto p-4 m-4">

1.tr[12]表示了哪一段区间的和？

<v-click>

$tr[12]=a[9]+a[10]+a[11]+a[12];$
</v-click>

2.tr[43]表示了哪一段区间的和？

<v-click>

$tr[43]=a[43];$
</v-click>

3.tr[1024]表示了哪一段区间的和？

<v-click>

$tr[1024]=a[1]+a[2]+\cdots+a[1023]+a[1024];$
</v-click>

</div>

---
id: 2
---

# 树状数组-如何拼成一段前缀的和?

那么比如说对于1到43这一段前缀来说

$tr[43]=a[43]$

$tr[42]=a[41]+a[42]$

$tr[40]=a[33]+a[34]+a[35]+\cdots+a[40]$

$tr[32]=a[1]+a[2]+a[3]+a[4]+\cdots+a[32]$

那么前缀 $[1, 43]$ 的和就是 $tr[43]+tr[42]+tr[40]+tr[32]$，原本$43$次加法运算，就被简化成了$4$次。

你会发现上面的过程可以总结成下面的形式。
- 需要前缀 $x$ 的和
- 得到区间 $tr[x] = [x-lowbit(x)+1, x]$ 的和
- $x = x-lowbit(x)$
- $x$ 为 $0$ 的时候终止运算，否则返回第一步

---
id: 2
---

# 树状数组-如何修改某个点的值

要考虑修改，我们就要考虑所有包含我们修改的点的 $tr$ 值。

来看下面这张图，这张图阐述了一种父子关系，树状数组上每个点的**和**由他的所有**儿子的和**组成。

<img src="/BIT.svg" class="h-40 mx-auto m-4"/>

所以，我们只需要从某个要修改的点出发，一直往父亲节点走，同时修改当前的树状数组节点值即可。

<div class="rounded-md shadow-md border-2 h-20 mx-auto p-4 m-4">

所以如何向父亲节点转移呢？     **<v-click>加上当前的lowbit即可。</v-click>**

</div>

---
id: 3
---
根据之前的内容，可写出以下代码
<!-- <div class="i-ph-terminal-window"></div> -->

```cpp
using ll = long long;
const int N = 2e6+5;
ll n, a[N], tr[N];
//查询一个前缀i的和
ll query(int i) {
    ll res = 0;
    while(i > 0) {
        res += tr[i];
        i = i - lowbit(i);
    }
    return 0;
}
//原数组中第i个点加上c
void add(int i, ll c) {
    while(i <= n) {
        tr[i] = tr[i] + c;
        i = i + lowbit(i);
    }
}
```

---
id: 3
---

有同学可能就会问了。那你这个 $lowbit()$ 函数在哪里呢？

其实我们只需要把 $x$ 和 $-x$ 按位与起来即可。

```cpp
int lowbit(int x) {
  return x & -x;
}
```
<v-click>
<div class="rounded-md shadow-md border-2 h-60 mx-auto p-4 m-4">
<div class="i-ph-pencil float-left p-3"></div>
<div class="font-semibold text-xl ">lowbit的原理</div>
<hr>

设原先 `x` 的二进制编码是 `(...)10...00`，全部取反后得到 `[...]01...11`，加 `1` 后得到 `[...]10...00`，也就是 `-x` 的二进制编码了。这里 `x` 二进制表示中第一个 `1` 是 `x` 最低位的 `1`。

`(...)` 和 `[...]` 中省略号的每一位分别相反，所以 `x & -x = (...)10...00 & [...]10...00 = 10...00`，得到的结果就是 `lowbit`。

可以不求甚解，记下即可。

</div>
</v-click>

---
id: 3
---

# 完整代码

```cpp {6-23|24-40|all}{lines:true,maxHeight:'380px'}
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 1e6+5;
ll n, q, a[N];
struct BIT {
	vector<ll> sum;
	BIT(int n) {
		sum.resize(n + 1, 0);
	}
	int lowbit(int x) {return x & -x;}
	void add(int k, ll x) {
		for( ; k <= n; k += lowbit(k)) sum[k] += x;
	}
	ll query(int k) {
		ll res = 0;
		for( ; k; k -= lowbit(k)) res += sum[k];
		return res;
	}
	ll query(int l, int r) {
		return query(r) - query(l - 1);
	}
} ;
int main() {
	ios::sync_with_stdio(false);
	cin.tie(nullptr), cout.tie(nullptr);
	cin >> n >> q;
	BIT DS(n);
	for(int i = 1; i <= n; i ++) {
		cin >> a[i]; DS.add(i, a[i]);
	}
	while(q --) {
		ll opt, l, r;
		cin >> opt >> l >> r;
		if(opt == 1) DS.add(l, r);
		else cout << DS.query(l, r) << endl;
	}
	
	return 0;
}
```