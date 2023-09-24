# 二枚のboard a, bを重ね合わせて、理想形xを作る

問題：https://atcoder.jp/contests/abc307/tasks/abc307_c

解説：https://atcoder.jp/contests/abc307/editorial/6690

提出：https://atcoder.jp/contests/abc307/submissions/45926221

```c++
int main () {
    // 設問に書いてある範囲
    int h = 10, w = 10;

    // 最後の評価をbitsetでやりたいけど、定数で記述する必要がある
    // 900 = 30 * 30
    //     = (h * 3) * (w * 3)
    bitset<900> boardx, boardc;


    // 以下、入力さえ合えば全自動


    // 上下左右の余白
    int marginh = h, marginw = w;
    // 余白を合わせた全体のサイズ
    int hall = h + 2 * marginh, wall = w + 2 * marginw;

    // 入力:a
    int ha, wa;
    cin >> ha >> wa;
    vector<vector<int>> boarda(ha, vector<int>(wa));
    rep(i, ha) rep(j, wa)
    {
        char c;
        cin >> c;
        if(c == '#') boarda.at(i).at(j) = 1;
    }

    // 入力:b
    int hb, wb;
    cin >> hb >> wb;
    vector<vector<int>> boardb(hb, vector<int>(wb));
    rep(i, hb) rep(j, wb)
    {
        char c;
        cin >> c;
        if(c == '#') boardb.at(i).at(j) = 1;
    }

    // 入力:x
    int hx, wx;
    cin >> hx >> wx;
    rep(i, hx) rep(j, wx)
    {
        char c;
        cin >> c;
        if(c == '#') boardx.set(wall * (marginh + i) + marginw + j, 1);
    }

    // 計算量はo((9hw)^2)
    rep(i, hall - (ha - 1)) rep(j, wall - (wa - 1))
    {
        rep(s, hall - (hb - 1)) rep(t, wall - (wb - 1))
        {
            // aは左上を(i, j) bは左上を(s, t)に配置
            //これをcに転記
            boardc.reset();
            rep(ii, ha) rep(jj, wa)
                boardc.set
                (
                    (i + ii) * wall + (j + jj),
                    boarda.at(ii).at(jj)
                );
            rep(ss, hb) rep(tt, wb)
                boardc.set
                (
                    (s + ss) * wall + (t + tt),
                    boardc[(s + ss) * wall + (t + tt)] | boardb.at(ss).at(tt)
                );

            // 評価
            if(boardx == boardc)
            {
                cout << "Yes" << endl;
                return 0;
            }
        }
    }

    cout << "No" << endl;

}
```

