要求在一棵树中找到两个点对应的最近公共祖先
暴力的算法就是直接逐级向上递增，直接去找，这样会使得时间复杂度非常大
因此考虑到倍增的思想去优化
首先用一个数组去把倍增出来的祖先进行存储，也就是2^0 2^1 2^2 这样的祖先进行记录
然后对于要进行查询的两个点，先算出这两个点对应的深度差异
然后把深度进行二进制优化一下，对应就是挑选二进制中的1进行跳跃，这样很方便实现查找
当出现在同一个深度时，再次进行两个点同步向上查找，最后找到共同祖先
luogu P3379
```cpp
const int K = 30;//2^30差不多就能覆盖到所有范围
vector<vector<int>>l1;//邻接矩阵
int up[500005][31];//倍增数组
int dist[500005];//深度
int n;
void DFS(int n,int p) {//LCA开始之前的初始化
    up[n][0] = p;
    rep(i, 1, K) {
        up[n][i] = up[up[n][i - 1]][i - 1];//倍增递归式
        //本质上是要想找到2^i的祖先,那么回头去找2^i-1的点然后在这个点上面去找2^i-1的点
        //这样写效率就能明显提升
    }
    for (auto it : l1[n]) {//对应的子节点也进行相关处理
        if (it == p) continue;
        dist[it] = dist[n] + 1;//深度处理
        DFS(it, n);
    }
}
int LCA(int a,int b) {
    if (dist[a] < dist[b]) {//保证a的深度大于b
        swap(a, b);
    }
    int diff = dist[a] - dist[b];//深度
    for (int i = K - 1; i >= 0; i--) {
        if (diff & (1 << i)) {//对深度进行二进制处理
            a = up[a][i];//处理完后进行跳跃
        }
    }
    if (a == b) {//如果正好上面就是b,直接返回
        return a;
    }
    for (int i = K - 1; i >= 0; i--) {
        if (up[a][i] != up[b][i]) {//如果当前对应的节点不对,那么再次向上进行跳跃
            a = up[a][i];
            b = up[b][i];
        }
    }
    return up[a][0];//这时候两个节点的父节点是一个点,直接返回
}
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    int n, m, s;
    cin >> n >> m >> s;
    l1.resize(n + 10);
    rep(i, 1, n - 1) {
        int a, b;
        cin >> a >> b;
        l1[a].push_back(b);
        l1[b].push_back(a);
        //进行建边,注意建双重边 这样避免最后的不出来对应的祖先的边
    }
    DFS(s, s);
    rep(i, 1, m) {
        int a, b;
        cin >> a >> b;
        cout << LCA(a, b) << '\n';
    }
}
```
关于其中深度二进制处理的内容
```cpp
int diff = dist[a] - dist[b];//深度
    for (int i = K - 1; i >= 0; i--) {
        if (diff & (1 << i)) {//对深度进行二进制处理
            a = up[a][i];//处理完后进行跳跃
        }
    }
```
这一部分，首先看到关于
(1<<i)
这一部分，这是把1的二进制向左移i位，也就是相当于检测diff的第i位上面是否是1，如果是那么直接跳跃，否则进行下一步计算
于此同时LCA还有一个公式，计算两个点之间的距离等同于这两个点的深度之和减去二倍的他们最近公共祖先的深度
