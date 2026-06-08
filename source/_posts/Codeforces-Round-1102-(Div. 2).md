---
title: Codeforces Round 1102 Div.2 补题记录
date: 2026-06-08 20:00:00
categories:
  - 题解
  - Codeforces
tags:
  - 补题
  - Codeforces
  - Div2
---

本文记录 Codeforces Round 1102 Div.2 的补题过程，包括题意理解、思路推导、错因分析和代码复盘。

<!-- more -->

# Codeforces Round 1102 (Div. 2)

### A

思路：

就从大到小排序，然后对每个位置判断题目条件即可，合法答案就是前两个最大值

```cpp
void sol(){
    cin >> n;
    vector<int> a(n);
    for(auto &x : a) cin >> x;
    
    sort(all(a), greater());

    for(int i = 2; i < n; ++ i) {
        if(a[i] != a[i-2] % a[i-1]) {
            cout << "-1\n";
            return;
        }
    }

    cout << a[0] << ' ' << a[1] << '\n';
}
```

### B

思路：

由于 $b | 12$，则 $a = n (mod 12)$，此时对 $a$ 取 $12$ 以内的数就基本就是答案了

设 $r = n \% 12$，若 $r < 10$ 或 $r = 11$，则自身构成回文，直接取 $a = r$，若 $r = 10$，只有 $n \ge 22$ 时取 $22$ 才能是回文的，否则无解

$b$ 直接取 $n - a$ 就够了

```cpp
void sol(){
    cin >> n;

    int r = n % 12, a, b;

    if(r <= 9 || r == 11) a = r;
    else {
        if(n >= 22) a = 22;
        else {
            cout << "-1\n";
            return;
        }
    }

    cout << a << ' ' << n - a << '\n';
}
```

### C && F

思路：

先理清题意，对于条件 $max(w_i, w_{i\%n + 1}) > h_i$，则 $w_i = w_{i\%n + 1}$ 表示的是，对于最后取值的 $w$ 相邻位置大于 $h_i$ 限制时就只能相等，等于多少是没有限制的，

所以用人话理解这个条件就是，相邻位置不相等只有最大值 $\le h_i$ 才成立，否则必须相等 

而题意就是求对于每个位置 $i$，限定为 $w_i = 0$ 时下能分配合法的 $w_i$ 的和最大是多少

由于包含 $0$ 的相邻位置必然不等，因此设 $0$ 位置为 $i$，相邻位置分别是 $i - 1, i + 1$，那么要求 $max(w_{i-1}, w_i) \le h_{i-1}, max(w_i, w_{i + 1}) \le h_i$，由 $w_i = 0$，又要最大化总和，那么相邻位置的取值其实只能固定为 $w_{i-1} = h_{i-1}, w_{i + 1} = h_{i+1}$，而对于其他位置则没有不相等的限制，那么就可以从两边传播看每个位置取值最大是多少且使得 $0$ 左右两个位置的值能保持，这个地方也可以手玩，从 $w_{i+1}$ 往右边传播直到 $w_{i}$，从 $w_{i-1}$ 往左边传播直到 $w_i$，两边都分别走了 $n - 1$ 步且是这个方向上的最大值分别记作 $mx1[j], mx2[j]$，对于中间的每个位置 $j$，结果的 $w_j = min(mx1[j], mx2[j])$，然后求和就是 $ans[i]$ 了，

暴力就是直接维护左右跑两遍拿 $mx$，C 到这里就够了

```cpp
void sol(){
    cin >> n;
    vector<int> a(n + 1), ans = a;
    for(int i = 1; i <= n; ++ i) cin >> a[i];

    for(int i = 1; i <= n; ++ i) {
        vector<int> w(n + 1), mx1(n + 1), mx2(n + 1);
        int id = i;
        for(int j = 1, cur = 0; j < n; ++ j) {
            int nid = id % n + 1;
            cur = max(cur, a[id]);
            mx1[nid] = cur;
            id = nid;
        }
        
        id = i;
        for(int j = 1, cur = 0; j < n; ++ j) {
            int pid = id - 1;
            if(pid == 0) pid = n;
            cur = max(cur, a[pid]);
            mx2[pid] = cur;
            id = pid;
        }
        
        int sum = 0;
        for(int j = 1; j <= n; ++ j) sum += min(mx1[j], mx2[j]);
        ans[i] = sum;
    }

    for(int i = 1; i <= n; ++ i) cout << ans[i] << " \n"[i == n];
}
```

接下来考虑上面这个东西如何预处理到 $O(1) \sim O(log {n})$ 能做（以下做法是赛后补题觉得妙有必要写题解）

注意到，$O(n^2)$ 做法中每次都是 $O(n)$ 去拿到每个点的最大值，这个地方可以通过删除全局最大值的一条边，将环变成链

此时，每个点的答案就直接是左右两边分别找前缀最大值求和得到的

为什么删除全局最大的边之后就能直接确定每个点 $mx$ 的取值了？由于 $s$ 到 $x$ 在环上有两条路径，$x$ 点的取值是两者最大值的 $min$，那么删掉最大值的那条边，即删除一条路径，剩下的那条路径的最大值一定会是最后 $x$ 这个点取到的 $mx$；这个地方的 $trick$：环上两条路径，通过删除支配性元素，剩下的取值固定

接下来问题变成了，固定起点 $s$，如何得到左右两边前缀最大值之和？显然 $dp$ 解决这个问题

由于最大值不具备可加性，需要单调栈优化，因为一个新的 $h[s]$ 可能可以更新 $s, s + 1 ... s + n$ 的所有值的最大值，必须跳过小于 $h[s]$ 的所有点才有效

以右边前缀最大值为例，$F[s] = max(h[s]) + max(h[s],h[s+1]) + \dots + max(h[s]...h[s-1]) = (p-i) * h[s] + F[p]$，设 $p$ 为 $h[p] \ge h[s]$ 的第一个位置

左边同理，

代码里面注意这个环变成链的映射关系怎么写，就结束了这个题

~~鉴定完毕：这个题最大的难点就是这个删边的trick，其他的都很典，菜完了~~

```cpp
void sol(){
    cin >> n;
    vector<int> a(n + 1);
    int mx = 0;
    for(int i = 1; i <= n; ++ i) cin >> a[i], mx = max(mx, a[i]);

    int cut = -1;
    for(int i = 1; i <= n; ++ i) if(a[i] == mx) {cut = i;break;}

    vector<int> ver(n + 1), b(n + 1);
    for(int i = 1; i <= n; ++ i) {
        ver[i] = (cut + i - 1) % n + 1;
        if(i < n) b[i] = a[ver[i]]; // b[i] 连接 x, x + 1 的边
    }
    
    vector<int> L(n + 1), R(n + 1, n), stk;
    for(int i = 1; i <= n; ++ i) {
        while(stk.size() && b[stk.back()] < b[i]) stk.pop_back();
        if(stk.size()) L[i] = stk.back();
        stk.push_back(i);
    }

    stk.clear();
    for(int i = n; i >= 1; --i) {
        while(stk.size() && b[stk.back()] < b[i]) stk.pop_back();
        if(stk.size()) R[i] = stk.back();
        stk.push_back(i);
    }
    
    vector<int> F(n + 1), G(n + 1);
    for(int i = n - 1; i >= 1; -- i) {
        int p = R[i]; // 右边第一个 >= b[i] 的位置
        F[i] = 1ll * (p - i) * b[i] + F[p];
    }

    for(int i = 1; i < n; ++ i) {
        int p = L[i]; // 左边第一个 >= b[i] 的位置
        G[i] = G[p] + 1ll * (i - p) * b[i];
    }

    vector<int> ans(n + 1);
    for(int i = 1; i <= n; ++ i) {
        ans[ver[i]] = F[i] + G[i - 1];
    }

    for(int i = 1; i <= n; ++ i) cout << ans[i] << " \n"[i == n];
}
```



### D

思路：

题意基本就是给两个数字，每次用相邻异或生成中间的，经过 $k$ 次生成 $2^k + 1$ 的数组，求结果数组中每个字符串内 $c1 \cdot c0$ 的和

那么用字符手玩就会发现，最后生成的只有 $a, b, a\oplus b$ 三种字符串，所以只要分别计算这三种字符串有多少个就够了

如果再模拟 $k = 1, 2, 3, 4, 5$ 这几个小状态的时候，就会发现一个惊人的结论，所有 $a, b, a \oplus b$ 各自的下标位置按照 $\%3$ 分组的

所以只要根据初始 $id_a = 1, id_b = 2^k + 1$ 这两个确定 $a, b$ 所属的组，$a\oplus b$ 所属的组别就是剩下那个，且由于 $b = 2^k + 1$ 所以与 $a$ 必然属于不同组

然后就直接计算一下 $2^k + 1$ 内 $\% 3 = 0, 1, 2$ 的数字有多少个就够了，这个就用总的除 $3$ + 剩下的就是了

然后就没了，~~一度怀疑这种题真的是 D？~~

```cpp
int cal(string s) {
    int c1 = count(all(s), '1');
    return 1ll * c1 * (n - c1);
}

// 2^k + 1 对 3 取模后，x 这个字符有多少个
int CNT(int k, int x){
    x %= 3;
    int tot = (1ll << k) + 1;
    int res = (tot) / 3;
    int rem = tot % 3;
    if(rem) {
        if(x <= rem && x) ++ res;
    }
    return res;
}

void sol(){
    cin >> n >> k >> a >> b;
    string c;
    for(int i = 0; i < n; ++ i) {
        char ch = ((a[i]-'0') ^ (b[i]-'0')) + '0';
        c.push_back(ch);
    }

    // cerr << c << '\n';

    int id = 1, id2 = ((1ll << k) + 1), id3 = ((id + id2) / 2);
    int cnt1 = CNT(k, 1), cnt2 = CNT(k, id2 % 3), cnt3 = CNT(k, id3 % 3);
    cerr << id << ' ' << id2 << ' ' << id3 << '\n';
    
    int ans = 
    1ll * cnt1 * cal(a) + 
    1ll * cnt2 * cal(b) + 
    1ll * cnt3 * cal(c);

    cout << ans << '\n';
}
```

