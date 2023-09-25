# n個の配列の内、上位k個の和を求める（入れ替えあり）

問題：https://atcoder.jp/contests/abc306/tasks/abc306_e

解説：https://atcoder.jp/contests/abc306/editorial/6607

提出：https://atcoder.jp/contests/abc306/submissions/45949331

スニペットに `bms` として登録済み

```c++
class MyBalancedMultiSet {
public:
    int n, k; // nが全体サイズ、kが上位サイズ
    multiset<int> x, y; // xが上位、yが下位
    ll s; // top（つまりx）のsum

    MyBalancedMultiSet(int size, int top_size)
    {
        n = size;
        k = top_size;

        rep(i, n)
        {
            if(i < k) x.insert(0);
            else y.insert(0);
        }
        s = 0;
    }

    void balance()
    {
        // xの要素数がkになるまでyから最大値をxに押し込む
        // ついでに、xに入ることになるのでsに足す
        while((int)x.size() < k)
        {
            auto iy = y.end(); iy--;
            x.insert((*iy));
            s += (*iy);
            y.erase(iy);
        }

        // yが空になればすること無し
        // （xが空ということがあるか？）
        if(x.empty() || y.empty()) return;

        // 以後、xにもyにも要素があるとき
        while(true)
        {
            // xの最小値とyの最大値を取り出しておく
            auto ix = x.begin();
            auto iy = y.end(); iy--;
            int ex = (*ix);
            int ey = (*iy);

            // xの最小値の方が大きかったら特に何もしなくてよい
            if(ex >= ey) break;

            // yの最大値の方が大きかったら下剋上が起こる
            s += (ey - ex); // sに変動がある
            x.erase(ix); // xから最小値がyに転落
            y.erase(iy); // yから最大値がxに格上げ
            x.insert(ey);
            y.insert(ex);
        }
    }

    void add(int v)
    {
        y.insert(v);
        balance();
    }

    // xまたはyからvを消す
    void erase(int v)
    {
        // xの中でvを探す
        auto ix = x.find(v);
        if(ix != x.end())
        {
            // 見つかれば答えsに影響がある
            s -= v;
            x.erase(ix);
        }
        else
        {
            // 見つからなければyから消す
            y.erase(y.find(v));
        }
    }
};

int main () {
    int n, k;
    cin >> n >> k;
    vector<int> a(n);

    MyBalancedMultiSet bms = MyBalancedMultiSet(n, k);

    int q;
    cin >> q;
    rep(i, q)
    {
        int p, w;
        cin >> p >> w;
        p--;
        bms.erase(a.at(p));
        bms.add(w);
        a.at(p) = w;
        cout << bms.s << endl;
    }
}

```



