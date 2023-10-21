# ベルトコンベア上を流れる商品に1つずつ印字する

問題：https://atcoder.jp/contests/abc325/tasks/abc325_d

解説：https://atcoder.jp/contests/abc325/editorial/7448

提出：https://atcoder.jp/contests/abc325/submissions/46836592

```c++
int main () {
    int n;
    cin >> n;
    vector<ll> t(n), d(n);
    vector<P> ps(n);
    rep(i, n) cin >> t.at(i) >> d.at(i);
    rep(i, n) ps.at(i) = P(t.at(i), t.at(i) + d.at(i));

    sort(all(ps));

    priority_queue<ll, vector<ll>, greater<ll>> q;

    int idx = 0;
    int ans = 0;
    ll time = 0;

    // 貪欲法
    while(true)
    {
        if(q.empty())
        {
            // もうこれ以上商品が残っていなければループ抜け
            if(idx == n) break;

            // 商品が残っていれば、時間をその商品の入る時間まで飛ばす
            time = ps.at(idx).first;
        }

        // -----------------------------------------------------------------

        // 印刷機の範囲内に入っている商品を追加する（厳密には出る時間を優先度付きキューに入れる）
        while(idx < n and ps.at(idx).first == time)
        {
            q.push(ps.at(idx).second);
            idx++;
        }

        // 印刷機の範囲から出たものを消す
        while(!q.empty() && q.top() < time) q.pop();

        // -----------------------------------------------------------------

        // 上記の処理後、印刷機の範囲内に商品が残っていたら、それに印刷して(++ans)消す(q.pop())
        if(!q.empty()) { q.pop(); ++ans; }

        // 時間を1マイクロ秒進める
        time++;
    }

    cout << ans << endl;



}
```

