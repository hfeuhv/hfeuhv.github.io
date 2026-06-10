---
title: Educational Codeforces Round 191
date: 2026-06-10 09:30:00
categories:
  - 题解
  - Codeforces
tags:
  - 补题
  - Codeforces
  - Edu
---

本文记录 Educational Codeforces Round 191 的补题过程
<!-- more -->



# Educational Codeforces Round 191

#### A

思路：

显然就两个方案，B 这个人要么用 AI，要么不用，总速度是不同的，注意用的时候，等待时 A 也在行动，所以要将总量减掉就够了

```cpp
int Ceil(int a, int b) {
    if(a <= 0) return 0;
    return (a + b - 1) / b;
}

void sol(){
    int x, y, z;
    cin >> n >> x >> y >> z;

    int ans = min({Ceil(n, x + y), z + Ceil(n - (x * z), 10 * y + x)});
    cout << ans << '\n';
}
```

#### B

思路：

考虑简单构造方式 1...n 4 个连续串，然后将第二个和第三个 1...n 的串翻转，

此时 n 为偶数时完美满足条件，但是奇数时 (n+1)/2 这个数字一直在中间每次距离都相等，

那就特殊处理这个多出的数字，设为 n，然后 n-1 个偶数按照上面方式构造，多出的这个一个在开头，两个在结尾，

此时还差一个，现在距离是 $2n - 1, 1$，你发现 $n \ge 2$，此时最小奇数就是 $3$，那么直接在第一次循环的 $1$ 后面再放一个 $n$ 就好了，他和开头距离是 $2$

和后面距离也不会相等，同时不会破坏原本结构

```cpp
void sol(){
    cin >> n;
    
    int x = n;
    if(n & 1) --n;
    if(x & 1) cout << x << " ";
    cout << 1 << " ";
    if(x & 1) cout << x << " ";
    for(int i = 2; i <= n; ++ i) cout << i << " ";
    for(int i = n; i >= 1; -- i) cout << i << " ";
    for(int i = n; i >= 1; -- i) cout << i << " ";
    for(int i = 1; i <= n; ++ i) cout << i << " ";
    
    if(x & 1) cout << x << " ";
    if(x & 1) cout << x << " ";
    cout << '\n';
}
```

#### C

思路：

先考虑能配对的直接删掉就好了，但是可能删除之后，原本不配对的变成配对了，因为是子序列

那就反着看，要让合法子序列最短，假设不删除，就是找最长的非法子序列，剩下的就是最短合法了，

而非法子序列一定是 $)))((($ 的结构，我们此时就能枚举这个中间的分界点，找到最长非法子序列的长度

在此基础上删除，怎么删都一定会让合法子序列长度减少最多一对，因为如果删除之后还是能够配对，这次删除无效，跟最开始一样，那么不管怎么删都会无效

所以能删的就删了，删的是分界点左边的 $($，右边的 $)$

```cpp
// 最大化 )))((( 的位置 p
// 左边 (，右边 )，能删除就删除，合法的可能会越来越少
void sol(){
    cin >> n >> k >> s, s = " " + s;
    
    vector<int> pre(n + 1), suf(n + 2);
    for(int i = 1; i <= n; ++ i) pre[i] = pre[i - 1] + (s[i] == ')');
    for(int i = n; i >= 1; -- i) suf[i] = suf[i + 1] + (s[i] == '(');

    int pos = -1, mxc = 0;
    for(int i = 0; i <= n; ++ i) {
        if(pre[i] + suf[i + 1] > mxc) pos = i, mxc = pre[i] + suf[i + 1];
    }

    string ans(n + 1, '0');
    for(int i = 1; i <= n && k; ++ i) {
        if(i <= pos) {
            if(s[i] == '(') ans[i] = '1', -- k;
        }else{
            if(s[i] == ')') ans[i] = '1', -- k;
        }
    }

    for(int i = 1; i <= n; ++ i) cout << ans[i];
    cout << '\n';
}
```

#### D

思路：

大模拟，我写这种题写得少，还以为 cf 不会出这种这么简单出思路的题目，结果真是这样，赛时给我整不自信了，哎哎哎

我的模拟思路就是，先看不合法的，由于只能交换一次，那么就最多两种数字不是连续块的，否则怎么换都不可能

那么设当前没有形成连续快的数字是 $x, y$，

单独看 $x$，最后的结构一定是 $x$ 形成一段连续块，且是一堆 $x$ 中一个非 $x$ 和外部的一个 $x$ 交换后形成的，而且交换后所有数字都形成连通块

对于 $y$ 同理，按照上面思路模拟

由于 $x$ 长度固定且结构固定，那么我们就能枚举所有 $cnt[x]$ 的区间，如果存在一个非 $x$，那么就尝试那这个和外面交换，看看能不能让现有的所有非法的变成合法

这个过程滑窗预处理一些东西就能做了

~~好难写啊~~

```cpp
int a[Maxn];
map<int, set<int>> make(){
    map<int, set<int>> mp;
    for(int i = 1; i <= n; ++ i) mp[a[i]].insert(i);
    return mp;
}

// 判断一个数字 x 交换后是不是正好可行？
// 就是个数区间长度内 非 x 和 外部 x 换了之后能不能成
vector<int> todo;
map<int, set<int>> mp;

bool okVal(int x){
    auto st = mp[x];
    int len = *st.rbegin() - *st.begin() + 1;
    if(len != st.size()) return 0;
    return 1;
}

bool ck(int p1, int p2){
    int x = a[p1], y = a[p2];
    if(x == y) return 0;

    mp[x].erase(p1), mp[y].erase(p2);
    mp[x].insert(p2), mp[y].insert(p1);

    bool ok = 1;
    ok &= okVal(x);
    ok &= okVal(y);

    for(auto v : todo) {
        ok &= okVal(v);
    }

    mp[x].erase(p2), mp[y].erase(p1);
    mp[x].insert(p1), mp[y].insert(p2);

    return ok;
}

bool work(int x){
    auto st = mp[x];

    int cnt = st.size();
    vector<int> px(all(st)), pre(n + 1);
    set<int> vals;
    for(int i = 1; i <= n; ++ i) {
        pre[i] = pre[i-1] + (a[i] == x);
        if(a[i] != x) vals.insert(i);
    }

    for(int i = 1; i + cnt - 1 <= n; ++ i) {
        int r = i + cnt - 1;

        if(pre[r] - pre[i-1] != cnt - 1) continue;

        int p1 = -1;
        if(px.front() < i) p1 = px.front();
        else if(px.back() > r) p1 = px.back();

        if(p1 == -1) continue;

        auto it = vals.lower_bound(i);
        if(it == vals.end() || *it > r) continue;
        
        if(ck(p1, *it)) return 1;
    }
    
    return 0;
}

void sol(){
    cin >> n;
    for(int i = 1; i <= n; ++ i) cin >> a[i];
    mp = make();
    todo.clear();
    for(auto &[x, st] : mp) {
        int len = *st.rbegin() - *st.begin() + 1;
        if(len != st.size()) todo.push_back(x);
    }

    if(todo.size() > 2) {
        cout << "NO\n";
        return;
    }
    
    if(todo.empty()) {
        cout << "YES\n";
        return;
    }

    int ok = 1;
    for(auto v : todo) {
        ok &= work(v);
    }

    cout << (ok ? "YES\n" : "NO\n");
}
```

#### E1 && E2

思路：

就是先按照每行 1 的数量从小到大排序，则此时对应的位数一定是从高位往低位的

然后再用输入的 n 行生成 n 个数字看看能不能是 $1\sim n$ 的排列，不合法直接 0

剩下的合法情况，就直接是所有 $1$ 数量相等的行数的阶乘就够了

~~好简单？这真的是 E 吗，还是群 u 提示到位才这么顺的（~~

```cpp
// 按 1 的数量从少往多排序行，拼出来的数字是对应高位往低位的，这个玩意看能不能覆盖 1~n
// 对于相同数量的 1 的行，直接就是阶乘，方案数就是他们乘积

int Fac[Maxn];
void init(){
    Fac[0] = 1;
    for(int i = 1; i <= n; ++ i) Fac[i] = Fac[i-1] * i;
}
void sol(){
    cin >> n;
    init();
    int lg = 0;
    while((1ll << lg) <= n) ++ lg;

    vector a(lg, vector<int>(n + 5));
    for(int i = 0; i < lg; ++ i) {
        for(int j = 1; j <= n; ++ j) {
            char ch;
            cin >> ch;
            a[i][j] = (ch - '0');
            a[i][0] += a[i][j];
        }
    }

    sort(all(a));

    vector<int> vis(n + 5);
    for(int j = 1; j <= n; ++ j) {
        int val = 0;
        for(int bit = lg-1, i = 0; bit >= 0; -- bit, ++ i){
            val += (1ll << bit) * a[i][j];
        }
        if(val < 1 || val > n || vis[val]) {
            cout << "0\n";
            return;
        }
        vis[val] = 1;
    }

    int ans = 1;

    for(int i = 0, j; i < lg; ++ i) {
        j = i;
        while(j < lg && a[j][0] == a[i][0]) ++ j;
        int len = j - i;
        ans *= Fac[len];
        i = j - 1;
    }

    cout << ans << '\n';
}
```

