# 宿題をする

問題：https://atcoder.jp/contests/typical-algorithm/tasks/typical_algorithm_b

解説：https://atcoder.jp/contests/typical-algorithm/editorial/707

提出：https://atcoder.jp/contests/typical-algorithm/submissions/46837401

```c++
int main () {
    int n;
    cin >> n;
    vector<P> ps(n);
    rep(i, n) cin >> ps.at(i).first >> ps.at(i).second;

    sort(all(ps), [](const P &left, const P &right){
        if(left.second == right.second) return left.first < right.first;
        else return left.second < right.second;
    });

    int ans = 0;
    int time = 0;
    rep(i, n)
    {
        if(time <= ps.at(i).first)
        {
            ans++;
            time = ps.at(i).second + 1;
        }
    }

    cout << ans << endl;
    
}
```

