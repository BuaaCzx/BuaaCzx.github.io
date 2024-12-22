---
layout: post
title: Codeforces Round 882 (Div. 2)
categories: [XCPC]
tags: [XCPC]
slug: cf882
---  

Codeforces Round 882 (Div. 2)

# A
## 题目大意
一段数列的总贡献是所有相邻两项差的绝对值的总和。现有一段长度为 $n$ 的数列，可以把它分为 $k$ 段，求这k段的贡献之和的最小值。
## 思路
将一段数列分成两段，本质上是将其中一个相邻两项的贡献不计入其中。贪心，计算出所有相邻两项差的绝对值，排序后不计入前 $k$ 大的值即可。
## 代码
```cpp
int n, k, a[1010];
bool cmp(int x, int y) { return x > y; }
void solve() {
    vector<int> aa;
    int ans = 0;
    cin >> n >> k;
    rep(i, 1, n) { cin >> a[i]; }
    rep(i, 1, n - 1) {
        aa.pb(abs(a[i + 1] - a[i]));
        ans += abs(a[i + 1] - a[i]);
    }
    sort(aa.begin(), aa.end(), cmp);
    rep(i, 0, k - 2) { ans -= aa[i]; }
    cout << ans << endl;
    return;
}
```
# B
## 题目大意
可以将一个数列分为连续的几组，每一组的贡献为 $$f(l,r)=a_l \verb'&' a_{l+1} \verb'&' ... \verb'&' a_r$$ ，在使总贡献最小的所有分法中，最多可以分出多少组？
## 思路
考虑到与运算的性质，不难得出，如果一个数列的与和为 $0$，那么分出尽量多的与和为 $0$ 的子串即可；如果一个数列的与和不为 $0$，显然只有只分一组才能使总共献最小。
## 代码
```cpp
ll a[200010];
void solve() {
    ll n, sum, ans = 0;
    cin >> n;
    rep(i, 1, n) {
        cin >> a[i];
        sum = i == 1 ? a[i] : sum & a[i];
    }
    if (sum != 0) {
        cout << 1 << endl;
        return;
    }
    //    cerr << sum << endl;
    ll t = -1;
    rep(i, 1, n) {
 
        t = t == -1 ? a[i] : t & a[i];
        //     cerr << t << endl;
        if (t == sum) {
            t = -1;
            ans++;
        }
    }
    cout << ans << endl;
}
```
# C
## 题目大意
对于一个序列，可以进行任意次以下的添加操作：在序列的末端添加上某个后缀的异或和。求这个序列在经过任意次操作后可以得到的最大值是多少。
## 思路
根据异或的性质 $a \oplus a =0$， $a \oplus 0 =a$ 不难得出，可以添加在序列中的数一定是初始序列中的子段异或和，求最大子段异或和即可。  
求解时既可以用普适性的 01 字典树来求最大子段异或和，在本题数据量不大的情况下，前缀和一定是大于 $0$ 小于 $2^8$ 的数，因此也可以遍历前缀和数组来求解。
## 代码
遍历前缀异或和：
```cpp
void solve() {
    int n, sum, ans = 0, x;
    cin >> n;
    set<int> pre;
    pre.insert(0);
    rep(i, 0, n - 1) {
        cin >> x;
        sum = i == 0 ? x : sum ^ x;
        pre.insert(sum);
        for (auto it : pre) {
            ans = max(ans, sum ^ it);
        }
    }
    cout << ans << endl;
}
```
01 字典树：
```cpp
int trie[1 << 10 + 1][2], idx;
void insert(int x) {
    int p = 1;
    per(i, 7, 0) {
        int bit = (x >> i) & 1;
        if (!trie[p][bit]) {
            trie[p][bit] = ++idx;
        }
        p = trie[p][bit];
    }
}
int calc(int x) {
    int p = 1, ans = 0;
    per(i, 7, 0) {
        int bit = (x >> i) & 1;
        if (trie[p][!bit]) {
            p = trie[p][!bit];
            ans = (ans << 1) + 1;
        }

        else {
            p = trie[p][bit];
            ans <<= 1;
        }
    }
    return ans;
}
void init() {
    rep(i, 0, idx) { trie[i][0] = trie[i][1] = 0; }
    idx = 1;
    insert(0);
}
void solve() {
    init();
    int n, x, sum = 0, ans = 0;
    cin >> n;
    rep(i, 1, n) {
        cin >> x;
        sum = sum ^ x;
        insert(sum);
        ans = max(ans, calc(sum));
    }
    cout << ans << endl;
}
```
# D
## 题目大意
$s$ 是一个长度为 $n$ 的 01 串，我们将交换 $s$ 中某两个位置的数称作对于 $s$ 的一次操作。有 $m$ 个 $s$ 的子串 $t_1,t_2,...,t_m$， $t(s)$ 为这 $m$ 个子串按序连接在一起得到的 01 串。  
有 $q$ 次询问，每次询问将 $s$ 的第 $x$ 位翻转($0$ 变成 $1$，反之亦然)后，使 $t(s)$ 字典序最大需要进行的最小操作数。
## 思路
题目要求字典序最大，那么 $t(s)$ 最前面的 $1$ 应尽可能多。考虑给每个数一个排名，排名越靠前代表我们把它变为 $1$ 的优先级越大。令存在于 $t(s)$ 中的数的总数为 $k_0$，记录当前字符串 $s$ 中 $1$ 的个数 $k$，那么只要将排名在 $1$ − $min(k,{k_0})$ 之间的数全部变为 $1$，最终的字符串字典序就一定是最大的。  
每个数的排名可以在未排名的数中用二分查找找第一个大于等于 $l_i$ 的数，如果这个数比 $r_i$ 还要大，则切换到下一个区间 $i+1$。所有区间查找完后，如果仍然有未排名的数，要记得将它们依次排名一下。  
要使某段前缀的数全部变为 $1$，只需统计这段前缀中 $0$ 的个数，维护一下即可。维护过程中有很多细节需要考虑，调了很久才过去。
## 代码
```cpp
int n, m, q, cnt = 1, k, num;
string s;
set<int> a;
int main() {
    ios;
    cin >> n >> m >> q;
    vector<int> rank(n + 1, 0), id(n + 1, 0);
    rep(i, 0, n - 1) { a.insert(i); }
    cin >> s;
    rep(i, 0, n - 1) { k += s[i] == '1'; }
    rep(i, 0, m - 1) {
        int l, r;
        cin >> l >> r;
        l--, r--;
        auto p = a.lower_bound(l);
        while (p != a.end() && *p <= r) {
            rank[*p] = cnt;
            id[cnt] = *p;
            cnt++;
            auto del = p;
            p++;
            a.erase(del);
        }
    }
    int k0 = cnt - 1;
    while (!a.empty()) {
        rank[*a.begin()] = cnt;
        id[cnt] = *a.begin();
        cnt++;
        a.erase(a.begin());
    }
    int t = min(k, k0);
    rep(i, 0, n - 1) {
        if (rank[i] >= 1 && rank[i] <= t) {
            num += s[i] == '1';
        }
    }
    rep(i, 0, q - 1) {
        int x;
        cin >> x;
        x--;
        if (s[x] == '0') {
            s[x] = '1';
            if (rank[x] >= 1 && rank[x] <= min(k, k0))
                num++;
            k++;
            if (k <= k0 && s[id[k]] == '1')
                num++;
        } else {
            s[x] = '0';
            if (rank[x] >= 1 && rank[x] <= min(k, k0))
                num--;
            if (k <= k0 && s[id[k]] == '1')
                num -= 1;
            k--;
        }
        cout << min(k, k0) - num << endl;
    }
    return 0;
}
```
