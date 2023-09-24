# 各桁が狭義の単調減少になっている数字

問題：https://atcoder.jp/contests/abc321/tasks/abc321_c

解説：https://atcoder.jp/contests/abc321/editorial/7261

提出：https://atcoder.jp/contests/abc321/submissions/45899198

```c++
int main () {
    // 321数は全部で1022個しか無いので先に作る
    vector<ll> ans;

    // 0～9の一つずつ使えばよい
    // なので10桁のbitで全探索をする（ただし、0:数字を使わない、1:0しか使わないを除く）
    for(int i = 2; i < (1 << 10); i++)
    {
        ll x = 0;
        for(int j = 9; j >= 0; j--)
        {
            if(i & (1 << j))
            {
                x *= 10;
                x += j;
            }
        }
        ans.push_back(x);
    }
    sort(all(ans));

    // 入力を受け付けて回答を出力
    int k;
    cin >> k;
    cout << ans.at(k - 1) << endl;
}
```

