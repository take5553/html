# n曲あるなかからランダム再生したときの、x+0.5秒後に0曲目が流れている確率

問題：https://atcoder.jp/contests/abc323/tasks/abc323_e

解説：https://atcoder.jp/contests/abc323/editorial/7357

提出：https://atcoder.jp/contests/abc323/submissions/46416467

※mint使用。スニペット登録済み。

```c++
int main () {
    int n, x;
    cin >> n >> x;
    vector<int> t(n);
    rep(i, n) cin >> t.at(i);

    // 確率DP
    vector<mint> dp(x + 1);
    dp.at(0) = 1;
    // nの逆数を予め求めておく
    mint r = (mint)1 / (mint)n;

    // dp.at(i) : i秒目がちょうど曲の切れ目になっている確率
    // 遷移
    // dp.at(i) = (dp.at(i - j曲目の長さ) * (j曲目が選ばれる確率)) の和
    //          = (dp.at(i - j曲目の長さ) * (1/n))の和
    //          = (dp.at(i - j曲目の長さ))の和 * (1/n)
    rep1(i, x)
    {
        rep(j, n) if(t.at(j) <= i) dp.at(i) += dp.at(i - t.at(j));
        dp.at(i) *= r;
    }

    // 解答
    // ans = x + 0.5秒後に0曲目がかかっている
    //     = 0曲目がカバーする範囲内で曲の切れ目がある * 0曲目が選ばれる確率
    //     = dp.at(0曲目がカバーする範囲)の和 * (1/n)
    //     = dp.at(x - (0秒以上0曲目の長さ未満))の和 * r
    mint ans = 0;
    rep(i, t.at(0)) if(x - i >= 0) ans += dp.at(x - i);
    ans *= r;

    cout << ans << endl;
}
```

```c++
const int mod = 998244353;
class mint {
    long long x;
public:
    mint(long long x=0) : x((x%mod+mod)%mod) {}
    mint operator-() const { return mint(-x); }
    mint& operator+=(const mint& a) { if ((x += a.x) >= mod) x -= mod; return *this; }
    mint& operator-=(const mint& a) { if ((x += mod-a.x) >= mod) x -= mod; return *this; }
    mint& operator*=(const  mint& a) { (x *= a.x) %= mod; return *this; }
    mint operator+(const mint& a) const { mint res(*this); return res+=a; }
    mint operator-(const mint& a) const { mint res(*this); return res-=a; }
    mint operator*(const mint& a) const { mint res(*this); return res*=a; }

    mint pow(ll t) const { if (!t) return 1; mint a = pow(t>>1); a *= a; if (t&1) a *= *this; return a; }
    // for prime mod
    mint inv() const { return pow(mod-2); }
    mint& operator/=(const mint& a) { return (*this) *= a.inv(); }
    mint operator/(const mint& a) const { mint res(*this); return res/=a; }

    friend ostream& operator<<(ostream& os, const mint& m){ os << m.x; return os; }
};
```

