---
title: 2026 JLCPC
date: 2026-06-15 10:00:00
categories:
  - 题解
  - VP
tags:
  - 补题
  - VP
  - JLCPC
  - 2026
---

本文记录 2026 JLCPC 的补题过程
<!-- more -->

# 2026 JLCPC

#### 写在前面

赛时 vp 5 题银了

什么时候可以单挑省金啊，哎哎哎

赛后补了两题，发现很多题目跟题解思路一致，对上脑电波了，对于相同思路的部分，我就放代码了，不同才会有思路详解

请结合官方题解食用（



#### A 

就是一个曼哈顿路径，手玩小 $case$ 的时候就能发现 $k$ 为奇数的时候不可能

证明：最后的路径可以拼成矩形，周长一定是偶数，而 奇数 $\times$ 奇数 $\not=$ 偶数恒成立，所以不可能

而对于 $k$ 为偶数的情况，你继续多玩几个也会发现最小次数一定是 $3$，且只有 $n \times n$ 这个矩形可以包含这个 $3k$ 长度的矩形时才有解，否则不可能

这个判断条件就是 $3k \le 4n$，然后就没了

```cpp
void sol(){
    cin >> n >> k;

    if(k & 1) {
        cout << "-1\n";
        return;
    }

    if(k/2 * 3 <= 2 * n) {
        cout << 3 << '\n';
    }else {
        cout << "-1\n";
    }
}
```



#### B

nanani 



#### L

模拟（好丑啊）

```cpp
void sol(){
    vector<string> a(17);
    for(auto &s : a) cin >> s;

    int Cnt[20]{};
    for(int i = 0; i < 17; ++ i) {
        char ch = a[i][0];
        if(ch == 'S' || ch == 'B') continue;
        
        if(isdigit(ch)) Cnt[stoi(a[i])] ++;
        else {
            if(ch == 'J') {
                Cnt[11] ++;
            }else if(ch == 'Q') {
                Cnt[12] ++;
            }else if(ch == 'K') {
                Cnt[13] ++;
            }else if(ch == 'A'){
                Cnt[14] ++;
            }
        }
    }

    int ans = 0;
    for(int i = 3; i <= 14; ++ i) {
        int cur = 0;
        for(int j = i; j <= 14; ++ j) {
            if(Cnt[j]) ++cur;
            else break;
        }
        ans = max(ans, cur);
    }

    if(ans < 5) cout << 0 << '\n';
    else cout << ans << '\n';
}
```



#### J

神秘时限 1.5s 卡我 $n^2 log n$ 做法（

```cpp
void sol(){
    cin >> n;
    vector<int> a(n + 1);
    for(int i = 1; i <= n; ++ i) cin >> a[i];

    vector<vector<int>> B(n + 1), S(n + 1);
    int base = 0;
    for(int i = 1; i <= n; ++ i) {
        for(int j = 1; j < i; ++ j) {
            if(a[j] > a[i]) B[i].push_back(j);
            else if(a[j] < a[i]) S[i].push_back(j);
        }
        base += B[i].size();
    }

    int ans = base;
    vector<int> d(n + 1);
    // 枚举到 r 时，l 位置累计贡献了多少？
    for(int r = 1; r <= n; ++ r) {
        int add = 0;
        for(int l = r - 1; l >= 1; -- l) {
            if(a[l] < a[r]) ++ add;
            else if(a[l] > a[r] )-- add;
            d[l] += add;
            ans = max(ans, base + d[l]);
        }
    }

    cout << ans << '\n';
}
```



#### I

就是按照数值从大往小构造 L 形，关键点是最优化构造前缀最大数组 $b$，能放多少放多少是最优的

```cpp
void sol(){
    cin >> n;
    vector<int> Cnt(n + 1);
    for(int i = 1, x; i <= n * 2; ++ i) cin >> x, Cnt[x] ++;

    vector<pii> todo, raw;
    for(int x = n; x >= 1; -- x) if(Cnt[x]) raw.push_back({x, Cnt[x]});

    int sum = n;
    vector<int> need(n + 1);
    for(auto &[x, cc] : raw) {
        int cur = min(cc - 1, sum);
        need[x] += cur;
        sum -= cur;
    }

    for(int x = n; x >= 1; --x) if(need[x]) todo.push_back({x, need[x]});

    vector<int> b;
    sort(all(todo));
    for(auto &[x, cc] : todo) {
        for(int t = 0; t < cc; ++ t) b.push_back(x);
        Cnt[x] -= cc;
    }

    if(sum != 0 || b.size() != n) {
        cout << "No\n";
        return;
    }

    vector<int> a;
    multiset<int> mst;
    for(int x = 1; x <= n; ++ x) for(int t = 0; t < Cnt[x]; ++ t) mst.insert(x);
    
    auto Add = [&](int x) {
        auto it = mst.find(x);
        if(it == mst.end()) {
            return 0;
        }
        mst.erase(it);
        a.push_back(x);
        return 1;
    };

    if(!Add(b[0])) {
        cout << "No\n";
        return;
    }

    for(int i = 1; i < n && mst.size(); ++ i) {
        if(b[i] != b[i-1]) {
            if(!Add(b[i])) {
                cout << "No\n";
                return;
            }
        }else {
            a.push_back(*mst.begin());
            mst.erase(mst.begin());
        }
    }

    if(a.size() != n) {
        cout << "No\n";
        return;
    }
    
    vector<int> mx(n);
    mx[0] = a[0];
    for(int i = 1; i < n; ++ i) {
        mx[i] = max(mx[i-1], a[i]);
        if(mx[i] != b[i]) {
            cout << "No\n";
            return;
        }
    }

    cout << "Yes\n";
    for(int i = 0; i < n; ++ i) cout << a[i] << " \n"[i + 1 == n];
}
```



#### D

思路：

~~跟官解一致，但还是写一下~~

考虑维护第一次分新组的集合 $S$，对于一个新的元素 $x$，尝试加入这个 $S$ 中，如果 $S$ 的元素新增分组了，那么这个 $x$ 就新开一组，否则说明在已有的某个组中

这个已有的某组可以在 $S$ 中二分查询属于哪个组，对于 $S[l ... mid]$ 这些点集，询问加入 $x$ 之后会不会新开一组，没有新开说明 $x$ 就在这些组中

接下来问题变成了，如何得到集合 $S$ 中包含多少组？

由于询问是 $S$ 中包含的完整集合数量，那么记 $C(S) = n/k - Q(U - S)$，其中 $U$ 表示全集，$C(S)$ 就是集合 $S$ 中包含的元组数量，用全集减掉 $S$ 查询和总数作个差就得到了

这个转化可能是这个题最关键的，上面加入元素二分找组其实挺自然的（

```cpp
int Ask(vector<int> a){
    vector<int> vis(n + 1), b;
    for(auto &x : a) vis[x] = 1;
    for(int i = 1; i <= n; ++ i) if(!vis[i]) b.push_back(i);

    int m = b.size();
    cout << "? " << m;
    for(int i = 0; i < m; ++ i) cout << ' ' << b[i];
    cout << endl;

    int cnt = 0;
    cin >> cnt;

    return n/k - cnt;
}

void Ans(const vector<int> &a){
    cout << "!";
    for(int i = 1; i <= n; ++ i) cout << ' ' << a[i];
    cout << endl;
}

vector<int> make(const vector<int> &a, int l, int r) {
    vector<int> vec;
    for(int i = l; i <= r; ++ i) vec.push_back(a[i]);
    return vec;
}

void sol(){
    cin >> n >> k;

    vector<int> a, ans(n + 1);
    a.push_back(1);
    ans[1] = 1;

    int idx = 2, cur = 1;
    for(int i = 2; i <= n; ++ i) {
        auto b = a;
        b.push_back(i);
        int cnt = Ask(b);
        if(cnt == cur) {
            int siz = a.size();
            int lb = 0, rb = siz - 1, mid, id = rb;// 牢元素，找已有分组
            while(lb <= rb) {
                mid = lb + rb >> 1;
                vector<int> L = make(a, 0, mid), nL = L;
                nL.push_back(i);
                if(L.size() == Ask(nL)) rb = mid - 1, id = mid;// 在左边
                else lb = mid + 1;
            }
            ans[i] = ans[a[id]];
        }else {
            a = b;
            ans[i] = idx ++; // 新元素，给新分组
        }

        cur = cnt;
    }

    Ans(ans);
}
```



#### M

思路：

这个期望有意思

考虑 $l, r$ 区间的期望 $E[x]$ 如何算

相邻相等元素不会贡献段数，只有相邻不等才会

记 $P(x)$ 表示每种元素 $x$ 相邻相等的概率，即 $P(x) = \binom{C(len, 2)}{C(c[x], 2)}$，就是在 $len$ 中选两个，在 $x$ 中也选两个的概率

那么随机排列两个相邻位置，颜色相等概率就是 $P_{same} = \sum_x P(x)$，而颜色不等概率就是 $1 - P_{same}$

那么期望就是 $E = 1 + (len - 1) \cdot (1 - P_{same})$，其中 $1$ 是第一个位置一定产生 $1$ 段的贡献，剩下的就是每段产生贡献的概率和就是期望

这个式子展开可以写成 $E = len + 1 - \sum_x C[x]^2 / len$

那么对于查询只要知道这个 $S = \sum_x C[x]^2$ 就好了，这个就是颜色数量就变成了莫队板子了，

然后只要考虑加入一个 $x$，删除一个 $x$，$S$ 如何变化就够了，不赘述了，看代码吧

```cpp
ll qmi(ll a, ll k, int m){ll res = 1; while(k){if(k & 1) res = res * a % m; a = a * a % m; k >>= 1;} return res;}

ll inv(int x) {return qmi(x, Mod - 2, Mod);}

struct Query{
    int id, l, r;
};

int Inv[Maxn];
void sol(){
    cin >> n >> q;
    vector<int> a(n + 1);
    for(int i = 1; i <= n; ++ i) cin >> a[i];

    vector<Query> Q(q);
    for(int i = 0, l, r; i < q; ++ i) cin >> l >> r, Q[i] = {i, l, r};
    
    int sq = sqrt(n);
    sort(all(Q), [&](const Query &A, const Query &B){
        int blA = (A.l-1) / sq, blB = (B.l-1) / sq;
        if(blA != blB) return blA < blB;

        if(blA & 1) return A.r > B.r;
        else return A.r < B.r;
    });

    vector<int> Cnt(n + 1), ans(q);
    
    int cur = 0;

    auto Add = [&](int p){
        int x = a[p];
        cur = (cur + 2 * Cnt[x] % Mod + 1) % Mod;
        Cnt[x] ++;
    };

    auto Del = [&](int p) {
        int x = a[p];
        cur = (cur - 2 * Cnt[x] % Mod + Mod) % Mod;
        cur = (cur + 1) % Mod;
        Cnt[x] --;
    };

    int L = 1, R = 0;
    for(int i = 0; i < q; ++ i) {
        auto [id, l, r] = Q[i];
        
        while(L > l) Add(--L);
        while(R < r) Add(++R);
        while(L < l) Del(L ++);
        while(R > r) Del(R --);

        int len = r - l + 1, tmp = cur * Inv[len] % Mod;
        int val = (len + 1 - tmp + Mod) % Mod;

        ans[id] = val;
    }

    for(int i = 0; i < q; ++ i) cout << ans[i] << "\n";
}

signed main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    for(int i = 1; i < Maxn; ++ i) Inv[i] = inv(i);
    // init();

    // #ifndef ONLINE_JUDGE
    // freopen("in.txt", "r", stdin);
    // #endif

    int t = 1;
    cin >> t;
    while (t--) {
        sol();
    }

    return 0;
}
```



#### 写在最后

虽然退役了，网瘾还没戒（

最近拿了蓝桥 $cb$ 国一，但是没什么用，就图一乐了

