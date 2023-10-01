# 3つのポリオミノの敷き詰め可能性

問題：https://atcoder.jp/contests/abc322/tasks/abc322_d

解説：https://atcoder.jp/contests/abc322/editorial/7302

提出：https://atcoder.jp/contests/abc322/submissions/46146731

※ `MyBoard` クラスを使用（スニペット登録済み、後述）

```c++
int main () {
    MyBoard poa = MyBoard(4, 4); poa.stdinput(); 
    MyBoard pob = MyBoard(4, 4); pob.stdinput();
    MyBoard poc = MyBoard(4, 4); poc.stdinput();

    // aの回転は不要とし、b,cを回転させる
    rep(rb, 4)
    {
        pob.rotate();
        rep(rc, 4)
        {
            poc.rotate();

            // ずらせる量を計るため、塗られているエリアを確定させる
            poa.updateArea();
            pob.updateArea();
            poc.updateArea();

            // oxa: aのはみ出ない範囲でx軸方向のずらし量を1つずつ増加
            // oya: aのはみ出ない範囲でy軸方向のずらし量を1つずつ増加
            for(int oxa = 0 - poa.left; oxa < 4 - poa.right; oxa++)
            for(int oya = 0 - poa.top; oya < 4 - poa.bottom; oya++)
            // oxb: bのはみ出ない範囲でx軸方向のずらし量を1つずつ増加
            // oyb: bのはみ出ない範囲でy軸方向のずらし量を1つずつ増加
            for(int oxb = 0 - pob.left; oxb < 4 - pob.right; oxb++)
            for(int oyb = 0 - pob.top; oyb < 4 - pob.bottom; oyb++)
            // oxc: cのはみ出ない範囲でx軸方向のずらし量を1つずつ増加
            // oyc: cのはみ出ない範囲でy軸方向のずらし量を1つずつ増加
            for(int oxc = 0 - poc.left; oxc < 4 - poc.right; oxc++)
            for(int oyc = 0 - poc.top; oyc < 4 - poc.bottom; oyc++)
            {
                // コピーを取らないと、元の物がズレる
                MyBoard poao = poa, pobo = pob, poco = poc;

                // ずらし
                poao.offset(oya, oxa);
                pobo.offset(oyb, oxb);
                poco.offset(oyc, oxc);

                // まっさらのボードに各ポリオミノを重ねていく
                MyBoard ans = MyBoard(4, 4);
                rep(i, 4) rep(j, 4)
                {
                    ans.at(i, j) += poao.at(i, j);
                    ans.at(i, j) += pobo.at(i, j);
                    ans.at(i, j) += poco.at(i, j);
                }
                
                // 条件を満たすか確認
                bool clear = true;
                rep(i, 4) rep(j, 4)
                {
                    if(ans.at(i, j) != 1)
                    {
                        clear = false;
                        break;
                    }
                }

                if(clear)
                {
                    cout << "Yes" << endl;
                    return 0;
                }
            }
        }
    }

    cout << "No" << endl;
}
```

```c++
class MyBoard {
public:
    int h, w;
    int top = -1, bottom = -1, left = -1, right = -1;

    vector<int> board;

    MyBoard(int height, int width)
    {
        h = height;
        w = width;
        board.assign(h * w, 0);
    }

    int& at(int y, int x)
    {
        return board.at(y * w + x);
    }
    
    void stdinput()
    {
        rep(i, h) rep(j, w)
        {
            char c;
            cin >> c;
            if(c == '#') at(i, j) = 1;
        }
    }

    void print()
    {
        rep(i, h)
        {
            rep(j, w) cout << at(i, j);
            cout << endl;
        }
    }

    void updateArea()
    {
        int t = intinf, l = intinf, b = -intinf, r = -intinf;
        rep(i, h)
        {
            bool f = false;
            rep(j, w)
            {
                if (at(i, j) == 1)
                {
                    chmin(l, (int)j);
                    chmax(r, (int)j);
                    f = true;
                }
            }
            if (f)
            {
                chmin(t, (int)i);
                chmax(b, (int)i);
            }
        }

        top = t;
        left = l;
        bottom = b;
        right = r;
    }

    void offset(int y, int x)
    {
        vector<int> r(h * w);
        rep(i, h) rep(j, w)
        {
            int p = (i + y) * w + (j + x);
            if (p < 0 || h * w <= p)
                continue;
            r.at(p) = at(i, j);
        }
        board = r;
    }

    void tume()
    {
        updateArea();
        offset(-top, -left);
    }

    void rotate(bool ccw = false)
    {
        vector<int> r(h * w);
        if (ccw == false) rep(i, h) rep(j, w) r.at(j * h + (h - 1) - i) = at(i, j);
        else rep(i, h) rep(j, w) r.at(((w - 1) - j) * h + i) = at(i, j);
        board = r;
        int t = h; h = w; w = t;
    }

    void flip(bool x, bool y)
    {
        vector<int> r(h * w);
        rep(i, h) rep(j, w)
        {
            int p = y ? (h - 1) - i : i;
            int q = x ? (w - 1) - j : j;
            r.at(p * w + q) = at(i, j);
        }
        board = r;
    }
};
```

