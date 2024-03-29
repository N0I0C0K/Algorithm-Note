# ST 表

## 解决什么问题

```text
有N个数，M次询问，每次给定区间[L,R]，求区间内的最大值。
N<=1e5 ,M<=1e6
```

对于这样的问题, 用线段树(每次查询是$O(logN)$)已经不行了, 我们必须保证每次查询都是$O(1)$复杂度.
**也就是**ST 表用于查询次数特别大的情况.

## 原理

前置知识:

- 区间 dp
- 倍增算法

首先我们知道$max(a,b,c) = max(max(a,b),max(b,c))$
也就是一整个区间可以拆分成**两个有重叠的区间**.
定义$f(i,j)为从a[i]开始的2^j个数的最大值$
显然:

- $f(i,0) = a[i]$
- $f(i,j) = max(f(i,j-1), f(i+2^{j-1},j-1))$

不太理解看[浅谈 ST 表](https://www.luogu.com.cn/blog/zhouziheng666/qian-tan-st-biao)

## 应用

- 区间最值
- 只要满足上面那类状态转移方程都可以

## 模板

```c++
const int maxn = 100005;

int a[maxn] = {};
int lg[maxn] = {-1};     //预处理log_2_(x)
int mmax[maxn][50] = {}; //表示mmax[i][j]表示 a[i:i+2^j-1]区间内的最大值

int main()
{
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; ++i)
    {
        scanf("%d", a + i);
        lg[i] = lg[i >> 1] + 1;
        mmax[i][0] = a[i];
    }
    for (int i = 1; i <= lg[n]; ++i)
    {
        for (int j = 1; j + (1 << i) - 1 <= n; ++j)
        {
            mmax[j][i] = max(mmax[j][i - 1], mmax[j + (1 << (i - 1))][i - 1]);
        }
    }
    int l, r;
    while (m--)
    {
        scanf("%d%d", &l, &r);
        int len = lg[r - l + 1];
        printf("%d\n", max(mmax[l][len], mmax[r - (1 << len) + 1][len]));
    }
    return 0;
}
```

C++风格封装

```cpp
template <typename T>
class SparseTable {
  using VT = vector<T>;
  using VVT = vector<VT>;
  using func_type = function<T(const T &, const T &)>;

  VVT ST;

  static T default_func(const T &t1, const T &t2) { return max(t1, t2); }

  func_type op;

 public:
  SparseTable(const vector<T> &v, func_type _func = default_func) {
    op = _func;
    int len = v.size(), l1 = ceil(log2(len)) + 1;
    ST.assign(len, VT(l1, 0));
    for (int i = 0; i < len; ++i) {
      ST[i][0] = v[i];
    }
    for (int j = 1; j < l1; ++j) {
      int pj = (1 << (j - 1));
      for (int i = 0; i + pj < len; ++i) {
        ST[i][j] = op(ST[i][j - 1], ST[i + (1 << (j - 1))][j - 1]);
      }
    }
  }

  T query(int l, int r) {
    int lt = r - l + 1;
    int q = ceil(log2(lt)) - 1;
    return op(ST[l][q], ST[r - (1 << q) + 1][q]);
  }
};
```
