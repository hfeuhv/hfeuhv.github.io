---
title: 2026 JSCPC && GDCPC 补题记录
date: 2026-06-02 15:06:53
categories:
  - 题解
  - VP
tags:
  - 补题
  - JSCPC
  - GDCPC
---

# 2026 JSCPC & GDCPC

#### L

思路：

考虑一个数字的时候，奇数先手必胜，偶数先手必败

考虑两个数字的时候，设 $a \le b$，通过手玩可以发现，$a,b$ 奇偶性相同的时候，先手必败，否则先手必胜

证明：由于最后面临 $0, 1$ 状态的人胜，推广到 $x, x + 1$ 的状态必胜，如果 $x$ 是偶数，先手选 $x，0,1$ 局面结果不变，因为后手选择 $x + 1$，先手面临 $x, x$，后手还是面临 $x -1, x$， 与最开始后手选择 $x$ 的结果一致，但如果 $x$ 是奇数，拿先手就能选 $x + 1$，后手面临 $x, x$，且都是奇数，选择之后 $x-1, x$，$x-1$ 为偶数，变成了上面的情况

若初始 $b \not = a + 1$，那么先手操作之后就会直接把必胜局面 $x, x + 1$ 让给后手，肯定不优，所有两个人都会一直操作 $b$，直到哪一方碰到了 $x, x + 1$ 的局面就赢了

这种情况下，就是 $a, b$ 奇偶性不同的时候，先手必胜，否则先手必败，因为 $x, x+1$ 奇偶性不同，一轮操作 $b$ 只会改变 $2$，不改变奇偶性

再将两个数字的情况推广到 $n$ 个数字，由于与下标没有关系，于是先排序，

操作效果就是，选择一个数字变成 $x - 1$，后缀全部变成 $x$，前缀不动

因为后缀全部变成相同的数，所以如果后缀长度是偶数的时候对答案是没有影响的，先手操作后后缀完全相同，如果后手选择后缀中的某个数，那么先手可以模仿直到所有后缀都相同，且最终后手面对的局面不变，和之前的后手选 $x + 1$ 推断类似

考虑找到第一个位置 $i$，使得 $a[i-1]$ 与 $a[i]$ 的奇偶性不同，且后缀是一段偶数，那么先手就操作这个 $i$，会直接让后手进入必败局面，

因为此时无论后手怎么操作，都会生成一个与之前相同的局面，先手永远可以找到后缀长度偶数且奇偶性不同的相邻位置，然后就判断这个条件就结束了

证明：就是看必胜态一步就能走到必败态，必败态无论走哪一步结果都是必胜态

用这个思想手推就行了，代码就是判断这个必胜条件结束了

```cpp
void sol(){
    cin >> n;
    vector<int> a(n + 1);
    for(int i = 1; i <= n; ++ i) cin >> a[i];

    sort(all(a));

    if(n % 2 == 1 && a[1] % 2 == 1) {
        cout << "Insight\n";
        return;
    }

    int start = (n % 2 == 1 ? 3 : 2);
    for(int i = start; i <= n; i += 2) {
        if(a[i] % 2 != a[i - 1] % 2) {cout << "Insight\n"; return;}
    }

    cout << "Maya\n";
}
```

#### K

思路：

就是按 b 从小到大排序，初始化 b = a + 1 开始；

没有成功卖出点分为两类：一类是库存不够而没有卖出的点，另一类是库存足够但是没达到限制的卖出点

随着 b 的增大，第二类会越来越多，这种点可以直接不管，而每次产生贡献的只可能来自第一类点，对于每个 b，维护应该删除掉哪些点集，枚举每个被删除的点，交换右边第一个第一类的点

贡献的计算方式就是：初始化模拟一遍算到基础的 base，然后删除点 -vi，加入就 +vj，没了

然后这个题就结束了

```cpp
struct Query{
    int b, id;

    bool operator<(const Query &o) const{
        return b < o.b;
    }
};

void sol(){
    cin >> n >> m;
    vector<int> v(n + 1);
    for(int i = 1; i <= n; ++ i) cin >> v[i];
    cin >> q;
    vector<int> ans(q + 1);
    vector<Query> Q(q + 1);
    for(int i = 1, b; i <= q; ++ i) cin >> b, Q[i] = {b, i};
    sort(Q.begin() + 1, Q.end());

    multiset<pii> mst;
    set<int> st;
    int cnt = 0, sum = 0, B = m + 1;
    for(int i = 1; i <= n; ++ i) {
        if(v[i] <= m) {
            ++ cnt;
            sum -= v[i];
        }else if(v[i] >= B){
            if(cnt) {
                mst.insert({v[i], i});
                cnt --;
                sum += v[i];
            }else st.insert(i);
        }
    }

    if(cnt > 0) sum += 1ll * v[n] * cnt;

    for(int i = 1; i <= q; ++ i) {
        auto [b, id] = Q[i];
        while(mst.size()){
            pii P = *mst.begin();
            int x = P.first, p = P.second;
            // cerr << x << ' ';
            if(x >= b) break;
            mst.erase(mst.begin());
            sum -= x;
            auto it = st.upper_bound(p);
            if(it == st.end()) {
                sum += v[n];
                continue;
            }
            // cerr << p << ' ' << *it << '\n';
            sum += v[*it];
            mst.insert({v[*it], *it});
            st.erase(it);
        }
        ans[id] = sum;
    }

    for( int i = 1; i <= q; ++ i ) cout << ans[i] << " \n"[i == q];
}
```

