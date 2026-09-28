---
title: 牛客周赛 Round 163 
date: 2026-09-28 10:00:00
categories:
  - 题解
  - 牛客
  - 牛客周赛
tags:
  - 补题
---

本文记录 牛客周赛 Round 163 的补题过程

<!-- more -->

# 牛客周赛 Round 163 

## A

思路：

枚举三个字符找到与 $a, b$ 两个不同的即可

代码：

```cpp
string s;
char a, b;
void solve(){
    cin >> s >> a >> b;
    for(auto ch : s) {
        if(ch != a && ch != b){
            cout << ch;
            return;
        }
    }
}
```

## B

思路：

$2^k$ 被 $16$ 进制整除，那么要求 $16$ 进制转为 $2$ 进制后后 $k$ 位全部为 $0$

直接按这个找就可以了，注意不能直接转换，会增加 $4$ 的常数，被卡掉了

还是特判来做

```cpp
string s;
char a, b;
int n, k;

int get(char ch){
    if(isdigit(ch)) {
        return ch - '0';
    }
    return ch - 'A' + 10;
};

auto trans(int x){
    string res;
//     cerr << x;
    while(x) {
        res.push_back(char(x % 2 + '0'));
        x /= 2;
    }
    while(res.size() < 4) 
        res = "0" + res;
//     cerr << ":" << res << '\n';
    // reverse(all(res));
    return res;
}

void solve(){
    cin >> s >> k;
    n = s.size();
    int q = k / 4, r = k % 4;
    for(int i = 0; i < min(q, n); ++ i) {
        if(get(s[n-1-i]) != 0) {
            Out("NO");
            return;
        }
    }
    if(q >= n) {
        Out("YES");
        return;
    }
    if(r) {
        int x = get(s[n-1-q]);
        if(x & ((1 << r) - 1)) {
            Out("NO");
            return;
        }
    }
    Out("YES");
}
```

## C

思路：

$n$ 非常小，直接暴力枚举所有可能

复杂度 $O(n!)$

代码：

```cpp
string s;
int n, k, a[7], b[7];
 
int ok;
void dfs(int pos, vt(int) &vec, vt(int) &vis){
    if(ok)
        return;
    if(pos>n){
        do{
            int flg = 1;
            for(int i = 1; i <= n; ++ i) {
                if(gcd(a[i], b[i]) > 1)
                    flg = 0;
            }
            if(flg) {
                ok = 1;
                return;
            }
        }while(next_permutation(b+1,b+n+1));
        return;
    }
    for(int i = 1; i <= n; ++ i) {
        if(vis[i])
            continue;
        vis[i] = 1;
        vec.push_back(a[i]);
        dfs(pos+1, vec, vis);
        vec.pop_back();
        vis[i] = 0;
    }
}
 
void solve(){
    cin >> n;
    for(int i = 1; i <= n; ++ i)
        cin >> a[i];
    for(int i = 1; i <= n; ++ i)
        cin >> b[i];
    vt(int) vec, vis(n+1);
    dfs(1, vec, vis);
    Out(ok ? "Bob" : "Alice");
}
```

## D

思路：

要构造使得列数 $1$ 的数量差值 $\le 1$，

考虑如何将每行 $1$ 的分类给若干列，最简单的就是将 $1$ 循环填列

使得 $1$ 的分配集合按照 $1, 2, \dots, m, 1, 2, \dots, m, 1, \dots$ 这种规律排列

多出来的部分不会与其他列差值超过 $1$

代码：

```cpp
int n, m;
void solve(){
    cin >> n >> m;
    vt(int) a(n + 1);
    for(int i = 1; i <= n; ++ i)
        cin >> a[i];
    int now = 1;
    auto nxt=[&](int id){
        return now == m ? 1 : now + 1;
    };
    vt(vt(int)) grid(n + 1, vt(int)(m + 1));
    for(int i = 1; i <= n; ++ i) {
        for(int j = 1, id; j <= a[i]; ++ j, now = nxt(now))
            grid[i][now] = 1;
    }
    for(int i = 1; i <= n; ++ i) {
        for(int j = 1; j <= m; ++ j) {
            cout << grid[i][j];
        }
        cout << '\n';
    }
}
```

## E

思路：

本质上是判定两条直线是否严格相交，并判断方向

方向使用向量的叉积就能很简单的判定了

代码：

> 模板 from 竹林（华东理工大学）

```cpp
constexpr double EPS = 1e-10;
using T = long double;
struct Point
{
    T x = 0, y = 0;
    Point(T _x = 0, T _y = 0) : x(_x), y(_y) {}
    friend bool operator==(const Point &x, const Point &y) { return (abs(x.x - y.x) <= EPS && abs(x.y - y.y) <= EPS); }
    friend Point operator+(const Point &x, const Point &y) { return {x.x + y.x, x.y + y.y}; }
    friend Point operator-(const Point &x, const Point &y) { return {x.x - y.x, x.y - y.y}; }
    friend Point operator*(const Point &x, const T k) { return {x.x * k, x.y * k}; }
    friend T operator*(const Point &x, const Point &y) { return x.x * y.x + x.y * y.y; }
    friend T operator^(const Point &x, const Point &y) { return x.x * y.y - x.y * y.x; }
    int toleft(const Point &p) const
    {
        T ans = (*this) ^ p;
        return (ans > EPS) - (ans < -EPS);
    }
    T len2() const {return x * x + y * y;}
    T len() const {return sqrtl(len2());}
};
 
struct Line
{
    Point p, v;
    Line() {}
    Line(Point a, Point b) { p = a, v = b; }
    bool is_connect(const Line &a) const { return abs(v ^ a.v) > EPS; }
    int toleft(const Point &a) const { return v.toleft(a - p); } // 点 a 是不是在当前点的坐标
    Point inter(const Line &a) const // 两条直线的交点
    {
        assert(is_connect(a));
        return p + v * ((a.v ^ (p - a.p)) / (v ^ a.v));
    }
    // 点 a 到直线的最短距离
	T dis2(const Point &a) const {
        int cr = v ^ (a - p);
        return (cr * cr) / (v * v);
    }
    T dis(const Point &a) const {
        return sqrtl(dis2(a));
    }
};
 
struct Segment
{
    Point a, b;
    Segment() {}
    Segment(Point _a, Point _b) : a(_a), b(_b) {}
    // 点 p 是不是在直线上
    int is_on(Point p) const
    {
        if (p == a || p == b)
            return -1;
        return (b - a).toleft(p - a) == 0 && (p - b) * (p - a) < -EPS;
    }
    // 两条线段是否严格有交点：1 表示严格相交，0 表示不相交，-1 表示一个直线的端点在另一个直线上
    int is_inter(const Segment &s) const 
    {
        if (is_on(s.a) || is_on(s.b) || s.is_on(a) || s.is_on(b))
            return -1;
        const Line l{a, b - a}, ls(s.a, s.b - s.a);
        return l.toleft(s.a) * l.toleft(s.b) == -1 && ls.toleft(a) * ls.toleft(b) == -1;
    }
    
    // 点 p 到线段的最短距离
    T dis2(const Point &p) const {
        Point v = b - a;
        if(v * (p - a) <= 0)
            return (p - a) * (p - a);
        if(v * (p - b) >= 0) 
            return (p - b) * (p - b);
       	T cr = v ^ (p - a);
        return (cr * cr) / (v * v);
    }
    
    T dis(const Point &p) const {
        return sqrtl(dis2(p));
    }
};

int n, m;
void solve(){
    cin >> n;
    Point A, B, C, D;
    cin >> A.x >> A.y >> B.x >> B.y;
    Segment seg(A, B);
    int ans = 0;
    for(int i = 0; i < n; ++ i) {
        cin >> C.x >> C.y >> D.x >> D.y;
        Segment s(C, D);
        if(seg.is_inter(s) == 1) {
            Point AB = B - A, AC = C - A;
            if(AB.toleft(AC) < 0) ++ ans;
            else -- ans;
        }
    }
    Out(ans);
}
```

## F

思路：

将每个 $s$ 加入 $Tire$ 中，在结尾标记 $ed = id$ 就能知道用了哪个字符串

同时维护每个字符串的剩余数量

偏板子的一个题，注意是字符串 $s$ 匹配 $t$ 的前缀，不是 $s$ 的前缀匹配，不能在 $Tire$ 上计数

~~犯蠢 wa 了一次~~

代码：

```cpp
const int N = 5e5 + 10;
struct Node {
    int ch[26]{};
    int cnt, ed;
}tr[N];

int nodecnt, cnt[N];
void insert(const string &s, int c, int i){
    int u = 0;
    for(auto ch : s) {
        int x = ch - 'a';
        auto &v = tr[u].ch[x];
        if(v == 0) {
            v = ++ nodecnt;
        }
        u = v;
        // tr[u].cnt += c;
    }
    tr[u].ed = i;
    cnt[i] = c;
}

int qry(const string &t){
    int u = 0, res = 0;
    for(auto ch : t) {
        int x = ch - 'a', v = tr[u].ch[x];
        if(v == 0)
            break;
        u = v;
        if(tr[u].ed > 0 && cnt[tr[u].ed] > 0)
            res = tr[u].ed;
    }
    if(res > 0) 
        cnt[res] --;
    return res;
}

int n, q;
void solve(){
    cin >> n >> q;
    for(int i = 1, c; i <= n; ++ i) {
        string s;
        cin >> s >> c;
        insert(s, c, i);
    }
    while(q --) {
        string t;
        cin >> t;
        Out(qry(t));
    }
}
```

## G

思路：

先将排列转为置换环，可以直接得到若干个环，将每个环转化为区间，就是题目要的区间了

区间左右端点用环内的最大最小值表示

对于 $w_i$ 表示有多少个区间严格包含 $i$ 这个点，严格包含是 $L \le i < R$，这个可以直接差分得到

而 $s(l, r) \ge t$ 表示询问区间 $(l, r)$ 包含了至少 $t$ 个已有区间 $(L_j, R_j)$，这个可以通过滑窗维护当前窗口内有多少个合法区间

枚举每个右端点，找到最大的合法左端点后，所有的 $l' \le l$ 就都是合法的左端点了，

统计数量用 $map$ 计数就好了，在移动左端点的时候加入值

代码：

```cpp
const int N = 2e5 + 5;
int n, k, t, p[N], w[N];

void solve(){
    cin >> n >> k >> t;
    for(int i = 1; i <= n; ++ i)
        cin >> p[i];
    vt(int) vis(n + 1);
    vt(pii) segs;
    for(int i = 1; i <= n; ++ i) {
        if(vis[i])
            continue;
        // 对于一个环，随便找一个起点，然后遍历整个环即可
        int u = i, L = n + 1, R = 0;
        while(!vis[u]) {
            vis[u] = 1;
            L = min(u, L), R = max(R, u);
            u = p[u];
        }
        segs.push_back({L, R});
    }
    sort(all(segs));
    vt(int) st(n + 1), ed(n + 1);
    for(auto &[l, r] : segs) {
        w[l]++, w[r]--;
        st[r] = l, ed[l] = r;
    }
    unordered_map<int, int> mp;
    for(int i = 1; i <= n; ++ i)
        w[i] += w[i-1];
    int ans = 0;
    mp[w[0]]++;
    for(int r = 1, l = 1, cnt = 0; r <= n; ++ r) {
        if(st[r] && st[r] >= l) 
            ++ cnt;
        while(l < r && cnt >= t) {
            int tag = (ed[l] && ed[l] <= r);
            if(cnt - tag >= t) {
                cnt -= tag;
                ++ l;
                mp[w[l-1]]++;
            }else break;
        }
        if(cnt >= t) {
            int v = k - w[r];
            ans += mp[v];
        }
    }
    Out(ans);
}
```

