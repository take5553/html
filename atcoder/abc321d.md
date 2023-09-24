# 全てのi, jについてmin(a.at(i) + b.at(j) , p)の和

問題：https://atcoder.jp/contests/abc321/tasks/abc321_d

解説：https://atcoder.jp/contests/abc321/editorial/7266

提出：https://atcoder.jp/contests/abc321/submissions/45921822

```c++
int main () {
    int n, m; ll p;
    cin >> n >> m >> p;
    vector<ll> a(n), b(m);
    rep(i, n) cin >> a.at(i);
    rep(i, m) cin >> b.at(i);

    sort(all(b));
    auto ruib = rui(b);

    ll ans = 0;

    rep(i, n)
    {
        ll t = p - a.at(i);

        // 二分探索でt以上（=aと足せばp以上になるもの）のposを探す
        int lb = lowpos(b, t);

        // (極端な例:lb=0を思い浮かべながら)

        // lb番目までは (a+b0) + (a+b1) + ... + (a+b_lb)が必要
        // aがlb個、b0 + ... + b_lbは累積和で計算を省略
        ans += a.at(i) * lb;
        ans += ruib.at(lb);

        // lb番目以降はpをm - lb個足す
        ans += p * (m - lb);
    }

    cout << ans << endl;
}
```

