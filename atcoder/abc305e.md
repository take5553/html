# 多始点ダイクストラ（グラフ上の警備員と警備されている点）

問題：https://atcoder.jp/contests/abc305/tasks/abc305_e

解説：https://atcoder.jp/contests/abc305/editorial/6539

動画：https://www.youtube.com/watch?v=z6lWDb9fMd8

提出：https://atcoder.jp/contests/abc305/submissions/46240828

```c++
int main () {
    int n, m, k;
    cin >> n >> m >> k;

    // グラフ作成
    vector<vector<int>> to(n);
    rep(i, m)
    {
        int a, b;
        cin >> a >> b;
        a--; b--;
        to.at(a).push_back(b);
        to.at(b).push_back(a);
    }

    // ダイクストラ法でいう「各頂点に至るまでの移動コスト」
    // 今回は「各頂点に至ることができる警備員の最大ヘルス」
    vector<int> d(n, -1);

    // priority_queueは(最大ヘルス、警備員が立っている地点)のペア
    priority_queue<P> q;

    // 多始点を設定
    // 警備員の数だけqueueに追加することで多始点を表現
    rep(i, k)
    {
        int p, h;
        cin >> p >> h;
        p--;
        d.at(p) = h;
        q.emplace(h, p);
    }

    // ダイクストラ開始
    while(!q.empty())
    {
        auto p = q.top(); q.pop();
        int h = p.first, v = p.second;

        // 最初にd.at(v) = hで設定しているので、
        // d.at(v) != hということはd.at(v)は既にh以上の数が格納されているということ
        // d.at(v)はv地点における警備員の最大ヘルス数であり
        // 初期値から減ることは絶対に無い
        // 加えて、v地点から伸びる他の点についても調べる必要が無い
        if(d.at(v) != h) continue;

        // v地点から伸びる他の点について全部調べる
        for(int u : to.at(v))
        {
            // dは最大ヘルスなので、既に大きいヘルスが格納されていたら飛ばしてよい
            if(d.at(u) >= h - 1) continue;

            // h-1の方が大きい時に以下を実行
            d.at(u) = h - 1;
            q.emplace(h - 1, u);
        }
    }
    // ダイクストラ修了
    // dには「各頂点に至ることができる警備員の最大ヘルス」が格納されている

    vector<int> ans;
    rep(i, n) if(d.at(i) >= 0) ans.push_back(i + 1);

    cout << ans.size() << endl;
    print1d(ans);

}
```

