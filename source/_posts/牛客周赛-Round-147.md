---
title: 牛客周赛 Round 147
date: 2026-06-10 12:30:00
categories:
  - 题解
  - 牛客
  - 牛客周赛
tags:
  - 补题
  - 牛客周赛
  - 前后缀 dp
  - 完全背包
---

本文记录 牛客周赛 Round 147 的补题过程
<!-- more -->


# 牛客周赛 Round 147

#### A

直接统计出现次数为 1 的字符即可

```python
s = input()

cnt = [0] * 26
for ch in s:
    cnt[ord(ch)-ord('a')] += 1;

ans = 0
for i in range(26):
    if cnt[i] == 1:
        ans += 1

print(ans)
```



#### B

注意到最优结构是43434...那就直接按这个走就够了

```python
x = int(input())

ans = (x // 10) * 7
rem = x % 10 

if rem >= 5:
    ans += 4
    
print(ans)
```



#### C

注意到每次区间 +1，区间内部差值不会改变，只有端点处差值变化对和产生贡献，那么维护差值数组，

每次的贡献计算方式：-旧的 + 新的，新的值直接用题目的公式更新就好了

```cpp
void sol(){
    cin >> n >> q;
    vector<int> a(n + 1), d(n + 1);
    for(int i = 1; i <= n; ++ i) cin >> a[i];
    int sum = 0;
    for(int i = 1; i < n; ++ i) d[i] = ((a[i + 1] - a[i]) % 5 + 5) % 5, sum += d[i];

    auto cal = [&](int id, int v) {
        int old = d[id], nv = (d[id] + v + 5) % 5;
        int delta = -old + nv;
        d[id] = nv;
        return delta;
    };

//     cerr << sum << '\n';
    while(q--) {
        int l, r;
        cin >> l >> r;
        if(l > 1) sum += cal(l - 1, 1);
        if(r < n) sum += cal(r, -1);
        cout << sum << '\n';
    }
}
```



#### D

思路：

需要 $b_i/b_{i-1} = p$ 为质数，即 $b_{i-1} = b_i / p$，那么枚举 $b_i$ 的所有质因子就够了，接下来变成最长子序列了，同时记录状态

定义 $f[x]$ 表示以 $x$ 结尾的最长子序列长度，$fid[x]$ 表示以 $x$ 结尾的最长子序列的位置，$dp[i]$ 表示以 $i$ 位置结尾的最长长度， $pre[i]$：表示 $i$ 位置结尾的最长子序列的前一个位置

定义好这些东西之后就做一遍 DP，记录最长长度和结尾 $id$ 就好了

```cpp
vector<int> Factor(int x){
    vector<int> vec;
    int t = x;
    for(int i = 2; i * i <= t; ++ i) {
        if(x % i == 0) {
            vec.push_back(i);
            while(x % i == 0) x /= i;
        }
    }
    if(x > 1) vec.push_back(x);
    return vec;
}
 
void sol(){
    cin >> n;
    vector<int> a(n);
    int mx = 0;
    for(auto &x : a) cin >> x, mx = max(mx, x);
     
    vector<int> f(mx + 5), fid(mx + 5, -1), pre(n + 1, -1), dp(n + 1, 1);
 
    int ans = 1, ansId = -1;
    for(int i = 0, x; i < n; ++ i) {
        x = a[i];
 
        auto fac = Factor(x);
        for(auto p : fac) {
            int y = x / p;
            if(f[y] + 1 > dp[i]) {
                dp[i] = f[y] + 1;
                pre[i] = fid[y];
            }
        }
 
        if(dp[i] > f[x]) f[x] = dp[i], fid[x] = i;
        if(dp[i] >= ans) ans = dp[i], ansId = i;
    }
 
    vector<int> path;
    for(int cur = ansId; cur != -1; cur = pre[cur]) {
        path.push_back(a[cur]);
    }
    reverse(all(path));
 
    cout << ans << '\n';
    for(auto x : path) cout << x << ' ';
    cout << '\n';
}
```



#### E

思路：

考虑前后缀 dp，结果序列结构是 greet，invite 或 invite，greet

以 greet, invite 为例

那么维护 preA[i] 表示前 i 个位置最少操作次数生成 greet，sufB[i] 表示后 i 个位置生成 invite 的最少操作次数

转移：对于每个位置生成子串，就直接用两个指针跑就够了，如果字符相等就都移动，否则操作数量增加，具体看代码吧，比较清晰

然后枚举划分点，使得前缀生成 greet，后缀生成 invtie 的最小操作数，注意划分点可以在 0

invite，greet 同理，两个方案取个 min 就是答案了

```cpp
const string T1 = "greet", T2 = "invite";

pair<vector<int>, vector<int>> make(const string &T){
    int m = T.size();
    vector<int> pre(n + 1, inf), suf(n + 2, inf);
    pre[0] = m;
    for(int i = 1; i <= n; ++ i) {
        int cur = 0;
        for(int pj = m - 1, pi = i; pj >= 0; -- pj) {
            if(pi >= 1 && T[pj] == s[pi]) --pi;
            else ++ cur;
        }
        pre[i] = min(pre[i], cur);
    }

    suf[n + 1] = m;
    for(int i = n; i >= 1; --i) {
        int cur = 0;
        for(int pj = 0, pi = i; pj < m; ++ pj) {
            if(pi <= n && T[pj] == s[pi]) ++ pi;
            else ++ cur;
        }
        suf[i] = min(suf[i + 1], cur);
    }

    return mkr(pre, suf);
}

int work(const string &A, const string &B){
    auto [preA, sufA] = make(A);
    auto [preB, sufB] = make(B);
    
    int ans = inf;
    for(int i = 0; i <= n; ++ i) {
        ans = min(ans, preA[i] + sufB[i + 1]);
    }

    return ans;
}

void sol(){
    cin >> n >> s, s = " " + s;
    cout << min(work(T1, T2), work(T2, T1)) << '\n';
}
```



#### F

思路：

注意到 $x / a$ 的取值是稀疏的，设这个数字集合为 $S$

则问题变成在 $S$ 中选 $n$ 次数字，和恰好为 $k$ 的构造方案，$1 \le a \le x$

那么可以将 $k - n$，允许某些位置为 $0$，对应的 $a$ 就变成 $x$ 即可，问题就变成选择 $ \le n$ 个数字凑成 $k$ 是否可行

这个问题显然更简单，可以用完全背包做，记 $f[x]$ 表示凑出 $x$ 的最小选择次数，$pre[s]$：表示最小次数下凑出 $s$ 前的状态，$fid[s]$：表示最小次数下凑出 $s$ 选择的是那个是数

然后跑一遍完全背包，回溯方案这个题就结束了

复杂度 $O(k \cdot \sqrt{x})$

```cpp
// 凑 k - n，dp[s]：凑和为 s 的最小次数，记录状态的 DP
// 完全背包：枚举可选值，枚举值域 i, s
pair<vector<int>, vector<int>> make(){
    vector<int> valA, valC; // 取值 A，贡献 C，贡献不会超过 min(x, k)，不优化会 TLE 
    for(int c = 2; c <= min(x, k); ++ c) {
        int a = x / c;
        if(a == 0) continue;
        if(x / a == c) {
            valC.push_back(c - 1);
            valA.push_back(a);
        }
    }

    return mkr(valA, valC);
}

void sol(){
    cin >> n >> x >> k;
    
    if(k < n || k > n * x) {
        cout << "NO\n";
        return;
    }

    auto [valA, valC] = make();

    int need = k - n;
    // f[s]：值为 s 时最小操作数
    // pre[s]：和为 s 时的前一个状态 (和)
    // fid[s]：和为 s 时选择的最后一个数的位置
    vector<int> f(need + 1, INF), pre(need + 1, -1), fid(need + 1, -1);
    f[0] = 0;
    for(int i = 0; i < valC.size(); ++ i) {
        int c = valC[i];
        for(int s = c; s <= need; ++ s) {
            if(f[s - c] + 1 < f[s]) {
                f[s] = f[s - c] + 1;
                pre[s] = s - c;
                fid[s] = i;
            }
        }
    }

    if(f[need] > n) {
        cout << "NO\n";
        return;
    }

    int cur = need;
    vector<int> path;
    while(cur > 0) {
        path.push_back(valA[fid[cur]]);
        cur = pre[cur];
    }
    while(path.size() < n) path.push_back(x);

    cout << "YES\n";
    for(auto x : path) cout << x << ' ';
    cout << '\n';
}
```



