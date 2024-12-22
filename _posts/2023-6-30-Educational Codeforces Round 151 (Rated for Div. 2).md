---
layout: post
title: Educational Codeforces Round 151 [Rated for Div. 2]
categories: [XCPC]
tags: [XCPC]
slug: cfedu151
---  

Educational Codeforces Round 151 [Rated for Div. 2]

# A
## 题目大意  
输入 $n,k,x$，有 $1$ 到 $k$ 中除了 $x$ 的所有数，每个数可以取任意次，任意给出一种取法，让取出来的所有的数和为 $n$。
## 思路  
如果 $k=1$，则 $x=1$，显然不存在。  
如果 $k>1$ 且 $x \neq 1$，则用 $1$ 可以得到 $n$。  
那么，只需要考虑 $k>1$ 且 $x=1$ 的情况。  
如果 $k=2$ ，则只能用 $2$，这时当 $n$ 被 $2$ 整除时可以得到 $n$，否则不行。  
如果 $k \geq 3$，则：如果 $n$ 为偶数，用 $2$ 可以得到；如果 $n$ 为奇数，让它减 $3$ 变为偶数后也可以用 $2$ 得到；特殊判断如果 $n=1$ 则无法得到。  
赛时认为 $n$ 如果可以得到，一定可以由 $n / i$ 个 $i$ 和 $1$ 个 $n \% i$ 得到，可以被数据 ``7 3 1`` hack。
## 代码  
```cpp
void solve() {
    int n, k, x;
    cin >> n >> k >> x;
    if (x != 1) {
        cout << "YES\n" << n << endl;
        rep(i, 1, n) cout << "1 ";
        cout << endl;
        return;
    }
    if (k == 1) {
        cout << "NO\n";
        return;
    }
    if (x == 1 && n == 1) {
        cout << "NO\n";
        return;
    }
    if (k != 2) {
        if (n % 2 == 0) {
            cout << "YES\n" << n / 2 << endl;
            rep(i, 1, n / 2) cout << "2 ";
            cout << endl;
            return;
        }
        cout << "YES\n" << 1 + (n - 3) / 2 << endl;
        rep(i, 1, (n - 3) / 2) cout << "2 ";
        cout << "3 ";
        cout << endl;
        return;
    } else {
        if (n % 2 == 1)
            cout << "NO\n";
        else {
            cout << "YES\n" << n / 2 << endl;
            rep(i, 1, n / 2) cout << "2 ";
            cout << endl;
            return;
        }
    }
}
```
# B
## 题目大意
平面图上有三个点 $A,B,C$，两个人b和c起始在 $A$ 上，每次移动可以往上下左右四个方向移动一格。b和c想要在保证自己走最近的一条路回到 $B,C$ 的情况下，走共同的路多一些。问它们最多能共同走多少格路。
## 思路
他们都走最近的路，意味着它们只能从上/下，左/右中各选一个方向走。因为它们的起点相同，如果它们同上或同下，同左或同右，说明可以同走一段路。
## 代码
```cpp
void solve() {
    int x[3], y[3], ans = 1;
    rep(i, 0, 2) cin >> x[i] >> y[i];
    int dx[2] = {x[0] - x[1], x[0] - x[2]}, dy[2] = {y[0] - y[1], y[0] - y[2]};
    if (dx[0] * dx[1] > 0) {
        ans += min(abs(dx[0]), abs(dx[1]));
    }
    if (dy[0] * dy[1] > 0) {
        ans += min(abs(dy[0]), abs(dy[1]));
    }
    cout << ans << endl;
}
```
# C
## 题目大意
给出长度 $m$ ，字符串 $s$，两个长度为 $m$ 的由数字组成的字符串 $l,r$。问能否设计一个由数字 $0$ 到 $9$ 组成的长度为 $m$ 的密码，要求密码不能是字符串 $s$ 的子串（子串可以是不连续的），且密码的第 $i$ 位数在 $l_i$ 与 $r_i$ 之间。
## 思路
对于密码的第 $i$ 位，所有在 $l_i$ 与 $r_i$ 之间的数满足要求，它们构成一个集合 $S_i$。试图将字符串 $s$ 分为 $m$ 段，如果对于每个 $i$，都满足第 $i$ 段中包含 $S_i$ 中的所有元素，那么很显然找不到这样的密码。否则可以找到。
## 代码
```cpp
char s[300010], l[12], r[12];
int m;
void solve() {
    int p = 0, len;
    unordered_set<int> a;
    scs(s);
    len = strlen(s);
    scd(m);
    scs(l);
    scs(r);
    rep(i, 0, m - 1) {
        rep(j, l[i] - '0', r[i] - '0') { a.insert(j); }
        for (; p < len; p++) {
            if (a.empty())
                break;
            a.erase(s[p] - '0');
        }
    }
    if (a.empty())
        cout << "NO\n";
    else
        cout << "YES\n";
}
```
# D
## 题目大意
选手的初始rating是 $0$，给出它 $n$ 场比赛的rating变化值 $a_i$。  
rating变化的机制：当选手的rating达到 $k$ 之后，如果之后它的某次rating变化将使它的rating低于 $k$ 了，它的rating只会变为 $k$。  
问当 $k$ 取多少时，选手最后的rating会最高。
## 思路
显然答案应为 $\sum_{1}^{m} a_i$。假设在达到 $k$ 后，rating清零，之后的rating以 $k=0$ 计算，最后 ${rating} + k$ 即为选手最终的rating。因此，我们只需要试图处理出 $k=0$ 时每个后缀的rating，就能够解决问题。  
求每个后缀的rating值，用dp来解决。对于每个后缀，我们可以把它拆分成两段，前一段的和为负数，后一段的和及其所有前缀和均为正数，该后缀的值即为后一段的和。用一个变量 $w$ 来记录``前一段的和``，如果 $w$ 为正数，说明它应该被划到后一段中，``sufsum[i]=sufsum[i+1]+w，w=0`` ，如果 $w$ 为负数，那么将它继续累积起来，``sufsum[i]=sufsum[i+1]``。
## 代码
```cpp
long long a[300010], sufsum[300010], ans, k, maxx = -INF;
int n;
void solve() {
    k = 0;
    maxx = -INF;
    posi = nega = 0;
    cin >> n;
    rep(i, 1, n) {
        sufsum[i] = 0;
        cin >> a[i];
    }
    sufsum[n + 1] = 0;
    int w = 0;
    per(i, n, 1) {
        w += a[i];
        if (w > 0) {
            sufsum[i] = w + sufsum[i + 1];
            w = 0;
        } else {
            sufsum[i] = sufsum[i + 1];
        }
    }
    rep(i, 0, n - 1) {
        k += a[i];
        if (k + sufsum[i + 1] > maxx) {
            maxx = k + sufsum[i + 1];
            ans = k;
        }
    }
    cout << ans << endl;
}
```
