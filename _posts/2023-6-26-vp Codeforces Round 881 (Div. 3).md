---
layout: post
title: vp Codeforces Round 881 (Div. 3)
categories: [XCPC]
tags: [XCPC]
slug: cf881
---  

vp Codeforces Round 881 (Div. 3)

# A

## 题目大意

给定一个数列，对这个数列上的每个数染色，对于每种颜色，染色的花费是 $该颜色的最大数-最小数$ ，求染色总花费的最大值。

## 思路  

贪心，每种颜色只染两个数并使这两个数的差尽量大即可，可以排序后从左右两边同时进行染色。

## 代码  

```cpp
int n, a[55], l, r;
void solve() {
    long long ans = 0;
    cin >> n;
    rep(i, 0, n - 1) cin >> a[i];
    sort(a, a + n);
    l = 0;
    r = n - 1;
    while (l < r) {
        ans += (a[r--] - a[l++]);
    }
    cout << ans << "\n";
}
```

# B

## 题目大意

给定一个数列，每次操作可以使一个连续区间上的数变成它的相反数，求最少需要几次操作能使数列中正数最多和正数最多的个数。

## 思路

贪心，对所有连续的非正数区间进行操作即可。

## 代码

```cpp
int n;
long long a[200010];
void solve() {
    int cnt = 0, flag = 0;
    cin >> n;
    long long ans = 0;
    rep(i, 0, n - 1) {
        cin >> a[i];
        ans += abs(a[i]);
        if (a[i] < 0 && flag == 0) {
            cnt++;
            flag = 1;
        } else if (a[i] > 0)
            flag = 0;
    }
    cout << ans << " " << cnt << "\n";
}
```

# C

## 题目大意

给定一个按层次从上到下从左到右编号的完全二叉树，询问某个节点到根节点的路径上所有节点的编号和。

## 思路

 $id_{father} = id_{son}/2$ ，直接计算即可，注意数据范围，要开long long。

## 代码

```cpp
void solve() {
    long long x, ans = 0;
    cin >> x;
    while (x != 0) {
        ans += x;
        x /= 2;
    }
    cout << ans << "\n";
}
```

# D

## 题目大意

给定一颗树，树上某两个节点有两个苹果。苹果会一直往下掉，掉到叶子节点后将会离开树。 有序数对 $(x, y)$ 表示第一个苹果从叶子节点 $x$ 离开，第二个苹果从叶子节点 $y$ 离开。每次询问给出两个苹果所在的节点，求符合条件的有序数对数量。

## 思路

求出每个节点下的叶子节点数量，答案即为两个苹果所在节点下的叶子节点数量乘积。  注意树的输入方式，给出的是一个无向无环联通图，每条树枝的具体方向并没有给定，但已知了根 $1$ ，在遍历的时候需要dfs图。

## 代码

```cpp
vector<int> to[200010];
int n, q;
long long num[200010], gone[200010];

long long dfs(int x) {
    long long ans = 0, flag = 0;
    for (auto it = to[x].begin(); it != to[x].end(); it++) {
        if (!gone[*it]) {
            flag = 1;
            gone[*it] = 1;
            ans += dfs(*it);
        }
    }
    return num[x] = flag == 0 ? 1 : ans;
}
void solve() {
    cin >> n;
    rep(i, 0, n) {
        gone[i] = 0;
        num[i] = 0;
        to[i].resize(0);
    }
    rep(i, 0, n - 2) {
        int u, v;
        cin >> u >> v;
        to[u].pb(v);
        to[v].pb(u);
    }
    gone[1] = 1;
    dfs(1);
    cin >> q;
    rep(i, 0, q - 1) {
        int x, y;
        cin >> x >> y;
        cout << num[x] * num[y] << endl;
    }
    return;
}
```

# E

## 题目大意

给定一个初始全部为 $0$ 的数列，和 $m$ 个区间。  如果一个区间内 $1$ 的数量比 $0$ 多，那么称这个区间是好的。  对这个数列进行 $q$ 次操作，操作 $i$ 可以将第 $x_i$ 个数变成 $1$ 。求第几次操作后， $m$ 个区间内会存在至少一个好的区间。

## 思路

求最小的满足条件的值，可以二分答案。复杂度 $O((m+n)logq)$ 。

## 代码

```cpp
struct seg {
    int l, r;
} s[100010];
int x[100010];
int n, m, q;
bool judge(int p) {
    int a[100010] = {0}, sum[100010] = {0};
    rep(i, 1, p) a[x[i]] = 1;
    rep(i, 1, n) { sum[i] = sum[i - 1] + a[i]; }
    rep(i, 0, m - 1) {
        if (2 * (sum[s[i].r] - sum[s[i].l - 1]) > s[i].r - s[i].l + 1)
            return 1;
    }
    return 0;
}
void solve() {
    cin >> n >> m;
    rep(i, 0, m - 1) { cin >> s[i].l >> s[i].r; }
    cin >> q;
    rep(i, 1, q) { cin >> x[i]; }
    int l = 1, r = q, ans = -1;
    while (l <= r) {
        int m = (l + r) / 2;
        if (judge(m)) {
            ans = m;
            r = m - 1;
        } else
            l = m + 1;
    }
    cout << ans << '\n';
    return;
}
```

# F1

## 题目大意

一颗树的每个节点的值都为 $-1$ 或 $1$ ，初始存在一个编号为 $1$ ，值为 $1$ 的节点，之后有 $n$ 次操作。  
操作1：添加一个编号为 $n+1$ ，值为 $x$ 的节点在节点 $v$ 下， $n$ 为当前节点总数。
操作2：询问节点 $u$ 和节点 $v$ 之间是否存在一条子路径，使得这条子路径的总值为 $k$ 。（easy version中有 $u=1$ ）

## 思路

由于每个结点的值为 $±1$ ，则在一条路径中总值的变化是连续的，因此只要 $k$ 在这条路径的最大子段和和最小子段和之间，便一定可以取到。在建树的时候在线dp求每个节点到根节点路径的最大子段和和最小子段和即可。  
赛时因为对最大子段和不熟，写了很久并且因为敲错了一个变量最后没有通过。

## 代码

```cpp
int t_size;
struct Node {
    int maxx, minn, sum1, sum2;
} t[200010];

void solve() {
    t_size = 1;
    t[1].maxx = t[1].sum1 = 1;
    t[1].sum2 = t[1].minn = 0;
    int n;
    cin >> n;
    rep(i, 0, n - 1) {
        char op;
        cin >> op;
        if (op == '+') {
            int v, x;
            cin >> v >> x;
            t_size++;
            t[t_size].sum1 = max(x + t[v].sum1, 0);
            t[t_size].sum2 = min(x + t[v].sum2, 0);
            t[t_size].maxx = max(t[t_size].sum1, t[v].maxx);
            t[t_size].minn = min(t[t_size].sum2, t[v].minn);
        } else if (op == '?') {
            int u, v, k;
            cin >> u >> v >> k;
            if (t[v].minn <= k && t[v].maxx >= k)
                cout << "YES\n";
            else
                cout << "NO\n";
        }
    }
    return;
}
```

# F2

## 题目大意

同F1，但 $u$ 不一定为 $1$ 。

## 思路

<++>

## 代码
