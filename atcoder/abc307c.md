# 二枚のboard a, bを重ね合わせて、理想形xを作る

問題：https://atcoder.jp/contests/abc307/tasks/abc307_c

解説：https://atcoder.jp/contests/abc307/editorial/6690

提出：https://atcoder.jp/contests/abc307/submissions/46146628

※ `MyBoard` クラスを使用（スニペット登録済み、後述）

```c++
int main () {
    int ha, wa;
    cin >> ha >> wa;
    MyBoard boarda = MyBoard(ha, wa);
    boarda.stdinput();
    
    int hb, wb;
    cin >> hb >> wb;
    MyBoard boardb = MyBoard(hb, wb);
    boardb.stdinput();

    int hx, wx;
    cin >> hx >> wx;
    MyBoard boardx = MyBoard(hx, wx);
    boardx.stdinput();

    MyBoard boardy = MyBoard(30, 30);
    rep(i, hx) rep(j, wx) boardy.at(10 + i, 10 + j) = boardx.at(i, j);

    for(int oxa = 0; oxa <= 30 - wa; ++oxa)
    for(int oya = 0; oya <= 30 - ha; ++oya)
    for(int oxb = 0; oxb <= 30 - wb; ++oxb)
    for(int oyb = 0; oyb <= 30 - hb; ++oyb)
    {
        MyBoard boardc = MyBoard(30, 30);
        rep(i, ha) rep(j, wa) boardc.at(oya + i, oxa + j) = boarda.at(i, j);
        rep(i, hb) rep(j, wb) boardc.at(oyb + i, oxb + j) |= boardb.at(i, j);

        if(boardc.board == boardy.board)
        {
            cout << "Yes" << endl;
            return 0;
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

