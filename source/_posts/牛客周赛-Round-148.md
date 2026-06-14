---
title: 牛客周赛 Round 148
date: 2026-06-14 21:50:00
categories:
  - 题解
  - 牛客
  - 牛客周赛
tags:
  - 补题
  - 牛客周赛
---

本文记录 牛客周赛 Round 148 的补题过程
<!-- more -->



# 牛客周赛 Round 148

#### A

思路：

就是 $x, y$ 的距离

```cpp
void sol(){
    int x, y;
    cin >> x >> y >> k;
    cout << abs(x - y) << '\n';
}
```



#### B

思路：

直接按质因子分解的复杂度是 $O(\sqrt n) = 10^9$ 会 TLE，然后猜了个上限不会超过 $10^7$（评测机上限貌似是 $10^8$ ），就过了 

然后枚举质因子 $q$ 看另一个是不是完全平方数，没了

```cpp
const int lim = 1e7;
void sol(){
    cin >> n;

    int t = n;
    vector<int> todo;
    for(int i = 2; i * i <= min(lim, t); ++ i) {
        if(n % i == 0) {
            todo.push_back(i);
            while(n % i == 0) n /= i;
        }
    }

    for(auto p : todo) {
        int q = t / p;
        if(p * q != t) continue;
        int sq = sqrt(q);
        if(sq * sq != q) continue;
        cout << sq << ' ' << p;
        return;
    }

    // cout << "-1\n";
}
```



#### C

思路：

注意到，题目讲的是有点的坐标数，所以不需要考虑生成的数量问题（数数可能会很难做了）

先不考虑删除的问题，你会发现，$1$ 个 $1$ 生成的就是一个杨辉三角系数，长度就是 $k + 1$

那么每个位置 $i$ 贡献就是 $min(d, k + 1)$，其中 $d = a[i + 1] - a[i]$，显然扩展上限是距离

然后你模拟删除就会发现正好是前 $k - 1$ 个不能要了，

然后注意第一个位置没有贡献，最后一个位置贡献没有上限，然后没了

```cpp
void sol(){
    cin >> n >> k;
    vector<int> a(n);
    for(auto &x : a) cin >> x;

    if(k == 0) {
        cout << n << '\n';
        return;
    }

    if(n == 1) {
        cout << "0\n";
        return;
    }

    sort(all(a));

    int ans = 0;
    for(int i = 1; i + 1 < n; ++ i) ans += min(a[i + 1] - a[i], k + 1);
    ans += k + 1;

    ans -= (k - 1);
    cout << ans << '\n';
}
```





#### D

思路：

从 $l$ 构造到 $r$，显然，操作的最小代价是 $msb(l\oplus r)$，第一次取得数字在 $l$ 得基础上加上 $2^{msb}$，后面保留这位剩下的花费就不会超过 $2^{msb}$ 了

然后按照每次遇到不同的 $bit$ 位就改变一位就能恢复 $r$ 了

```cpp
void sol(){
    int a, b;
    cin >> a >> b;

    int cur = a;
    vector<int> ans;
    ans.push_back(a);
    for(int bit = 63; bit >= 0; -- bit) {
        int u = (a >> bit & 1), v = (b >> bit & 1);
        if(u != v) {
            cur ^= (1ll << bit);
            ans.push_back(cur);
        }
    }

    cout << ans.size() << '\n';
    for(auto x : ans) cout << x << ' ';
}
```



#### E

思路：

~~更搞笑的一题~~

显然按位独立考虑，只有偶数个 $1$ 可以通过操作消除，奇数个 $1$ 是不可能的

这个结论可以通过操作得到，这里的操作就是相邻位置相同的时候，可以全部变成 $0$，不同的时候则是相当于移动 $1$ 的位置 $+1/-1$，花费为 $1$（这个观察可能靠经验）

对于最小花费，就是看 $01$ 串每两个 $1$ 匹配的花费最小是多少，花费是相邻 $1$ 的距离

因为是环，就按照两个方向跑一遍匹配然后取二者的 $min$ 就完了，

其实我认为可能按照长度为 $n$ 的窗口算花费比较严谨，但是有点难写，遂随便猜了一下，就过了

然后就结束了

```cpp
int work(vector<int> b){
    int res = 0;
    for(int i = 0, lst = -1; i < n; ++ i) {
        if(b[i] == 0) continue;
        if(lst == -1) {
            lst = i;
        }else {
            res += i - lst;
            lst = -1;
        }
    }

    int idx = 0;

    auto next = [&](int id) {
        if(id == 0) return n-1;
        return id - 1;
    };

    int res2 = 0;
    for(int i = 0, lst = -1; i < n; ++ i, idx = next(idx)) {
        if(b[idx] == 0) continue;
        if(lst == -1) {
            lst = idx;
        }else {
            res2 += n - abs(lst - idx);
            lst = -1;
        }
    }

    return min(res, res2);
}

void sol(){
    cin >> n;
    vector<int> a(n);
    for(auto &x : a) cin >> x;

    int ans = 0;
    for(int bit = 30; bit >= 0; -- bit) {
        vector<int> b(n);
        for(int i = 0; i < n; ++ i) b[i] = (a[i] >> bit & 1);
        
        int cnt = count(all(b), 1);
        if(cnt & 1) {
            cout << "-1\n";
            return;
        }

        if(cnt == 0) continue;
        ans += work(b);
    }

    cout << ans << '\n';
}
```



#### F

思路：

对于完全二叉树，可以直接知道 $x$ 属于哪一层 $h$，也能知道总的层数 $H$

对于 $x$ 点作为中点的路径数量，可以考虑从 $x$ 开始往外扩展的点的距离 $d$

那么对于一个距离 $d$，贡献可以分为子树内部与子树内部匹配，子树内部和子树外部匹配（父亲）

记左右子树，父亲方向距离为 $d$ 的节点数量分别为 $L, R, U$，那么 $ans = 1 + \sum_d L \cdot R + (L + R) \cdot U$，这里的 $1$ 是 $d = 0$ 时的特殊情况

那么问题就是这三个数量怎么算了

对于 $L, R$ 子树方向的节点数量，完全二叉树有个很好的性质，就是能用层数 $O(1)$ 直接算到以 $u$ 为节点距离为 $d$ 的节点编号范围

对于父亲方向，往上怕 $t$ 步到达的祖先记为 $p$，那么如果 $x$ 属于 $p$ 的其中一颗子树，$p$ 的另一颗子树内的节点此时与 $x$ 内部距离为 $d$ 的节点直接配对就又能产生贡献

但是这里在 $p$ 使用的到子树的距离需要将 $x$ 到 $p$ 的距离减掉才行，这样对所有可达祖先节点求和就是祖先方向距离为 $d$ 的节点数量了

然后这题就没了，注意枚举可以直接暴力 $30+$，用不到 $H$，因为找不到祖先了自然不会算到

```cpp
int CntD(int u, int dep){
    if(u > n) return 0;

    int L = u << dep, R = min(n, ((u + 1) << dep) - 1);
    if(L > n) return 0;
    
    return R - L + 1;
}

int CntU(int x, int d){
    int res = 0;

    int cur = x;
    for(int t = 1; t <= d && cur > 1; ++ t) {
        int fa = cur / 2;
        if(t == d) {
            res ++;
        }else {
            int son = n + 1;
            if(cur == fa * 2) son = fa * 2 + 1;
            else son = fa * 2;
            res += CntD(son, d - t - 1);
        }
        cur = fa;
    }
    
    return res;
}

void sol(){
    cin >> n >> x;

    int ans = 1;
    for(int d = 1; d <= 35; ++ d) {
        int L = CntD(2 * x, d - 1), R = CntD(2 * x + 1, d - 1), U = CntU(x, d);
        ans += 1ll * L * R + 1ll * (L + R) * U;
    }

    cout << ans << '\n';
}
```

