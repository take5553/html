# 映画を見る

問題：https://atcoder.jp/contests/tessoku-book/tasks/math_and_algorithm_bn

解説：（鉄則本6.4）

提出：https://atcoder.jp/contests/tessoku-book/submissions/46836995

```c++
int main () {
    int n;
    cin >> n;
    vector<P> ps(n);
    rep(i, n) cin >> ps.at(i).first >> ps.at(i).second;

    // 貪欲法（終わりが速いもの順にソート）
    sort(all(ps), [](const P &left, const P &right){
        if(left.second == right.second) return left.first < right.first;
        else return left.second < right.second;
    });

    int time = 0;
    int ans = 0;

    // 順番に見れば良い
    rep(i, n)
    {
        if(time <= ps.at(i).first)
        {
            ans++;

            // 終わりまで拘束されるので時間を飛ばす
            time = ps.at(i).second;
        }
    }

    cout << ans << endl;
}
```

