# ドリンクと自動販売機

[オブジェクト指向エクササイズをやってみる \- Qiita](https://qiita.com/opengl-8080/items/6f0a458df9c34eccf76c)

↑をなぞる。Javaなので、それをあえてC#でやってみる。

## 基にするコード

クラス
[VendingMachine.cs](https://github.com/take5553/OOPPractice/blob/OriginalCode/OOPPractice/VendingMachine.cs)
[Drink.cs](https://github.com/take5553/OOPPractice/blob/OriginalCode/OOPPractice/Drink.cs)

顧客
[Form1.cs](https://github.com/take5553/OOPPractice/blob/OriginalCode/OOPPractice/Form1.cs)

## 出来上がったコード

[take5553/OOPPractice at Practice1](https://github.com/take5553/OOPPractice/tree/Practice1)

## やってみた

以下、参考サイトと同じような流れでやってみた。それぞれの変更ポイントだけを書いていくことにするので、同じようにC#でやってみようという人は、参考サイトを参考にしながら

1. とりあえず何も考えずに該当箇所を変える
2. 変更によって生じた不都合はエディタが教えてくれるので、エディタが文句言わなくなるところまで整える
3. 実際に起動してみて動かなかったらデバッグしてさらに整える

ということをしてほしい。

### 名前を省略しない

IDEの「変数の名前変更機能」を使えば簡単。でも省略してもいいもの（抽象度が高くて名前が付けにくいとか）は無理に名づけをしない、あるいは簡単な名前にする。

~~~C#
public Drink Buy(int i, int kindOfDrink)
{
    // 処理
}
~~~

引数の`i`は支払いとして使われているので`payment`に改名。

~~~C#
public Drink Buy(int payment, int kindOfDrink)
{
    // 処理
}
~~~

### 全てのプリミティブ型と文字列型をラップすること

このときに色々やってしまうと先に進まなくなる可能性がある。単純にラップするのがいいのではないか。

~~~C#
class VendingMachine
{
    int quantityOfCoke = 5;
    int quantityOfDietCoke = 5;
    int quantityOfTea = 5;
    int numberOf100Yen = 10;
    int change = 0;


    public Drink Buy(int payment, int kindOfDrink)
    {
~~~

ここに見えている`int`をすべて置き換える。

~~~C#
class VendingMachine
{
    Stock stockOfCoke = new Stock(5);
    Stock stockOfDietCoke = new Stock(5);
    Stock stockOfTea = new Stock(5);
    Stack<Coin> numberOf100Yen = new Stack<Coin>();
    List<Coin> change = new List<Coin>();


    public Drink Buy(Coin payment, DrinkType kindOfDrink)
    {
~~~

クラスを作成すると`List`や`Stack`が使えるところが見えてきて、次の「ファーストクラスコレクションを使用すること」を知っていると色々やりたくなってしまうけど、複数のことを同時に進めてしまうとややこしくなってしまうので、一旦ここではシンプルに`Stack`や`List`を使えばいいと思う。

#### `Stock`クラス

~~~C#
class Stock
{
    public Stock(int quantity)
    {
        this.Quantity = quantity;
    }

    public int Quantity { get; private set; }

    public void Decrement()
    {
        this.quantity--;
    }

}
~~~

#### `DrinkType`列挙型

~~~C#
enum DrinkType
{
    COKE,
    DIET_COKE,
    TEA
}
~~~

#### `Coin`クラス

~~~C#
class Coin
{
    public const int ONE_HUNDRED = 100;
    public const int FIVE_HUNDRED = 500;

    public Coin(int amount)
    {
        this.Amount = amount;
    }

    public int Amount { get; private set; }
}
~~~

### ファーストクラスコレクションを使用すること

できた`List<Coin>`や`Stack<Coin>`をクラスでラップする。

~~~C#
    Stack<Coin> numberOf100Yen = new Stack<Coin>();
    List<Coin> change = new List<Coin>();
~~~

↓

~~~C#
    StackOf100Yen numberOf100Yen = new StackOf100Yen();
    Change change = new Change();
~~~

これらをある意味プリミティブ型と考えれば「全てのプリミティブ型と文字列型をラップすること」の延長となる。

#### `StackOf100Yen`クラス

~~~C#
class StackOf100Yen
{
    private Stack<Coin> numberOf100Yen = new Stack<Coin>();

    public StackOf100Yen()
    {
        for (int i = 0; i < 4; i++)
        {
            numberOf100Yen.Push(new Coin(Coin.ONE_HUNDRED));
        }
    }

    public int Count()
    {
        return numberOf100Yen.Count;
    }

    public void Add(Coin coin)
    {
        numberOf100Yen.Push(coin);
    }

    public Coin Pop()
    {
        return numberOf100Yen.Pop();
    }
}
~~~

#### `Change`クラス

~~~C#
class Change
{
    private List<Coin> change;

    public Change()
    {
        change = new List<Coin>();
    }

    public Change(Coin coin)
    {
        change = new List<Coin>();
        change.Add(coin);
    }

    public Change(Change change)
    {
        this.change = new List<Coin>(change.change);
    }

    public void Add(Coin coin)
    {
        change.Add(coin);
    }

    public void Add(Change change)
    {
        this.change.AddRange(change.change);
    }

    public void Clear()
    {
        change.Clear();
    }

    public int GetAmount()
    {
        int ret = 0;
        foreach (Coin coin in change)
        {
            ret = ret + coin.Amount;
        }
        return ret;
    }
}
~~~

#### `VendingMachine`クラス

~~~C# 
class VendingMachine
{
    Stock stockOfCoke = new Stock(5);
    Stock stockOfDietCoke = new Stock(5);
    Stock stockOfTea = new Stock(5);
    StackOf100Yen numberOf100Yen = new StackOf100Yen();
    Change change = new Change();


    public Drink Buy(Coin payment, DrinkType kindOfDrink)
    {
        if((payment.Amount != Coin.ONE_HUNDRED) && (payment.Amount != Coin.FIVE_HUNDRED))
        {
            change.Add(payment);
            return null;
        }

        if((kindOfDrink == DrinkType.COKE) && (stockOfCoke.Quantity == 0))
        {
            change.Add(payment);
            return null;
        } else if ((kindOfDrink == DrinkType.DIET_COKE) && (stockOfDietCoke.Quantity == 0))
        {
            change.Add(payment);
            return null;
        } else if ((kindOfDrink == DrinkType.TEA) && (stockOfTea.Quantity == 0))
        {
            change.Add(payment);
            return null;
        }

        if(payment.Amount == Coin.FIVE_HUNDRED && numberOf100Yen.Count() < 4)
        {
            change.Add(payment);
            return null;
        }

        if (payment.Amount == Coin.ONE_HUNDRED)
        {
            numberOf100Yen.Add(payment);
        } else if (payment.Amount == Coin.FIVE_HUNDRED)
        {
            change.Add(calculateChange());
        }

        if (kindOfDrink == DrinkType.COKE)
        {
            stockOfCoke.Decrement();
        } else if(kindOfDrink == DrinkType.DIET_COKE)
        {
            stockOfDietCoke.Decrement();
        }
        else
        {
            stockOfTea.Decrement();
        }

        return new Drink(kindOfDrink);
    }

    private Change calculateChange()
    {
        Change ret = new Change();
        for (int i = 0; i < 4; i++)
        {
            ret.Add(numberOf100Yen.Pop());
        }
        return ret;
    }

    public Change Refund()
    {
        Change result = new Change(change);
        change.Clear();
        return result;
    }
}
~~~

### Getter、Setter、プロパティを使用しないこと

クラスが持つフィールドをGetterで外部に晒したら、外部で何に使われるか分からなくなり、将来的な変更の影響範囲が広がってしまうからそれを防ぐという考え方。

> - オブジェクトが持つ値を無加工で取得しているところ。
> - 呼び出し側で、取得した値を使って何らかのロジック（`if` 分岐や値の加工）を実行しているところ。
>
> この観点で修正していく。

顧客クラスがGetterを使ってやっていることを当該クラスに移譲する。

例えば

~~~C#
if((payment.Amount != Coin.ONE_HUNDRED) && (payment.Amount != Coin.FIVE_HUNDRED))
{
    // 100円または500円ではないとき
}
~~~

これは`coin`の`Amount`プロパティの値を顧客クラスが取り出して、`100`または`500`かどうかを判断しているけど、これを`coin`クラスにやらせる。

~~~C#
if ((coin.DoesNotEqual(Coin.ONE_HUNDRED) && (coin.DoesNotEqual(Coin.FIVE_HUNDRED))){
    // 100円または500円ではないとき
}
~~~

`coin.DoesNotEqual()`では何をやっているのかと言うと

~~~C#
public DoesNotEqual(Coin comparison)
{
    return this.amount != comparison.amount;
}
~~~

これで「データを使った処理をクラスに閉じ込める」ことを進めることになる。

#### `Coin`クラス

~~~C#
class Coin
{
    public static readonly Coin ONE_HUNDRED = new Coin(100);
    public static readonly Coin FIVE_HUNDRED = new Coin(500);

    private int amount;

    public Coin(int amount)
    {
        this.amount = amount;
    }

    public bool Equals(Coin comparison)
    {
        return this.amount == comparison.amount;
    }

    public bool DoesNotEqual(Coin comparison)
    {
        return this.amount != comparison.amount;
    }

    public Money ToMoney()
    {
        return new Money(this.amount);
    }
}
~~~

#### `Stock`クラス

~~~C#
class Stock
{
    private int quantity;

    public Stock(int quantity)
    {
        this.quantity = quantity;
    }


    public void Decrement()
    {
        this.quantity--;
    }

    public bool IsEmpty()
    {
        return quantity == 0;
    }

}
~~~

#### `StockOf100Yen`クラス

~~~C#
class StackOf100Yen
{
    private Stack<Coin> numberOf100Yen = new Stack<Coin>();

    public StackOf100Yen()
    {
        for (int i = 0; i < 4; i++)
        {
            numberOf100Yen.Push(new Coin(100));
        }
    }

    public void Add(Coin coin)
    {
        numberOf100Yen.Push(coin);
    }

    public bool DoesNotHaveChange()
    {
        return numberOf100Yen.Count < 4;
    }

    public Change TakeOutChange()
    {
        Change ret = new Change();
        for (int i = 0; i < 4; i++)
        {
            ret.Add(numberOf100Yen.Pop());
        }
        return ret;
    }
}
~~~

#### ファーストクラスコレクションで悩む

Getterが使えないとなると、ファーストクラスコレクションで問題が起きる。

具体的には`Stack<Coin>`型のフィールドを持つ`Change`クラスで、総額を計算して返すことができなくなった。

~~~C#
public int GetAmount()
{
    int ret = 0;
    foreach (Coin coin in change)
    {
        ret = ret + coin.Amount;
    }
    return ret;
}
~~~

お釣りの総額を計算するのは`Change`でよいが、`Coin`の金額を扱うのは`Coin`に限定されている。

ここで「お金」という概念をクラス化することを発見する。（というか一般概念をクラス化することは基本テクニック）

金額の足し算はお金クラスにやらせる。

#### `Money`クラス

~~~C#
class Money
{
    private int amount;

    public Money(int amount)
    {
        this.amount = amount;
    }

    public Money Add(Money money)
    {
        return new Money(this.amount + money.amount);
    }

    override public string ToString()
    {
        return amount.ToString();
    }
}
~~~

### 1つのクラスにつきインスタンス変数は2つまでにすること

インスタンス変数を性質ごとに分類することを促す。特に同じ型の物は連想配列でクラスに持たせてしまう。

~~~C#
class VendingMachine
{
    Stock stockOfCoke = new Stock(5);
    Stock stockOfDietCoke = new Stock(5);
    Stock stockOfTea = new Stock(5);
    StackOf100Yen numberOf100Yen = new StackOf100Yen();
    Change change = new Change();
~~~

↓

~~~C#
class VendingMachine
{
    Storage storage = new Storage();
    CoinMech coinMech = new CoinMech();
~~~

ただし、実務ではこんな制約を付けると逆に不自然になったりするかもしれないので、あくまでエクササイズ目的として。

#### `Storage`クラス

~~~C#
class Storage
{
    Dictionary<DrinkType, Stock> stocks = new Dictionary<DrinkType, Stock>();

    public Storage()
    {
        stocks.Add(DrinkType.COKE, new Stock(5));
        stocks.Add(DrinkType.DIET_COKE, new Stock(5));
        stocks.Add(DrinkType.TEA, new Stock(5));
    }

    public bool IsEmpty(DrinkType drinkType)
    {
        return stocks[drinkType].IsEmpty();
    }

    public void Decrement(DrinkType drinkType)
    {
        stocks[drinkType].Decrement();
    }
}
~~~

#### `CoinMech`クラス

~~~c#
class CoinMech
{
    private CashBox cashBox = new CashBox();
    private Change change = new Change();

    public CoinMech()
    {
        for (int i = 0; i < 10; i++)
        {
            cashBox.Add(new Coin(CoinType.ONE_HUNDRED));
        }
    }

    public void AddChange(Coin payment)
    {
        this.change.Add(payment);
    }

    public void AddChange(Change change)
    {
        this.change.Add(change);
    }

    public void AddCoinIntoCashBox(Coin payment)
    {
        this.cashBox.Add(payment);
    }

    public bool DoesNotHaveChange()
    {
        return this.cashBox.DoesNotHaveChange();
    }

    public Change TakeOutChange()
    {
        return this.cashBox.TakeOutChange();
    }

    public Change Refund()
    {
        Change result = new Change(change);
        change.Clear();
        return result;
    }
}
~~~

#### `CashBox`クラス

`StackOf100Yen`からのリネーム。

### else句は使わないこと

今回の一番難しいところ。これをきっかけとして自販機の責務まで見直している。そして結局責務移譲先のクラスでif文2つ作って「elseを書いていない詐欺」みたいな形になっている。

100円硬貨や500円硬貨は`Coin`クラスを継承して`OneHundredCoin`とか`FiveHundredCoin`とかにして、メソッドのオーバーロードでそれぞれに対応すればif文書く必要が無いんじゃないか。

と思ったらC#は静的型付けなので、都合よく解釈してくれない。つまり

~~~C#
private void DropCoinFromHolder(OneHundredCoin coin)
{
    // 処理
}

private void DropCoinFromHolder(FiveHundredCoin coin)
{
    // 処理
}
~~~

とかにして

~~~C#
private Coin coinHolder;

DropCoinFromHolder(coinHolder);
~~~

みたいなことはできない。つまり結局は「100円なのか、500円なのか」はif文（またはswitch構文）で判断するしかないっぽい。

ということは「else句は使わない」というのは、動的型付け言語ならさっき自分が考えたようなオーバーロードで型スイッチという発想を促すことに繋がるけど、静的型付け言語なら「無理やん」で終わってしまう可能性があるということか。

#### `OneHundredCoin`クラス

~~~c#
class OneHundredCoin : Coin
{
    public OneHundredCoin()
    {
        amount = 100;
    }
}
~~~

#### `FiveHundredCoin`クラス

~~~c#
class FiveHundredCoin : Coin
{
    public FiveHundredCoin()
    {
        amount = 500;
    }
}
~~~

#### `Coin`クラス

~~~c#
class Coin
{
    protected int amount;

    public Money ToMoney()
    {
        return new Money(this.amount);
    }
}
~~~

#### `CashBox`クラス

~~~C#
class CashBox
{
    private Stack<OneHundredCoin> stackOf100Yen = new Stack<OneHundredCoin>();
    private Stack<FiveHundredCoin> stackOf500Yen = new Stack<FiveHundredCoin>();


    public void Add(OneHundredCoin coin)
    {
        stackOf100Yen.Push(coin);
    }

    public void Add(FiveHundredCoin coin)
    {
        stackOf500Yen.Push(coin);
    }

    public bool DoesNotHaveChange()
    {
        return stackOf100Yen.Count < 4;
    }

    public Change TakeOutChange()
    {
        Change ret = new Change();
        for (int i = 0; i < 4; i++)
        {
            ret.Add(TakeOutOneHundredCoin());
        }
        return ret;
    }

    private OneHundredCoin TakeOutOneHundredCoin()
    {
        return stackOf100Yen.Pop();
    }

}
~~~

#### `CoinMech`クラス

~~~C#
class CoinMech
{
    private CashBox cashBox = new CashBox();
    private Payment payment = new Payment();

    public CoinMech()
    {
        for (int i = 0; i < 10; i++)
        {
            cashBox.Add(new OneHundredCoin());
        }
    }

    public void Put(Coin coin)
    {
        payment.Hold(coin);
    }

    public bool DoesNotHaveChange()
    {
        return this.cashBox.DoesNotHaveChange();
    }

    public void Commit()
    {
        payment.Commit(cashBox);
    }

    public Change Refund()
    {
        return payment.Refund();
    }
}
~~~

#### `Payment`クラス

~~~C#
class Payment
{
    private Change change = new Change();
    private Coin coinHolder;

    public void Hold(Coin coin)
    {
        coinHolder = coin;
    }

    public void Commit(CashBox cashBox)
    {
        switch (coinHolder)
        {
            case OneHundredCoin oneHundredCoin:
                cashBox.Add(oneHundredCoin);
                change = new Change();
                break;
            case FiveHundredCoin fiveHundredCoin:
                cashBox.Add(fiveHundredCoin);
                change = cashBox.TakeOutChange();
                break;
        }

        coinHolder = null;

    }

    public Change Refund()
    {
        Change result = new Change(change);
        change.Clear();
        return result;
    }
}
~~~

### すべてのエンティティを小さくすること

本来は

* 1ファイル（つまり1クラス）50行まで
* 1パッケージ10ファイルまで

という制約のことだけど、[Rule of 30 – When is a Method, Class or Subsystem Too Big? \- DZone Java](https://dzone.com/articles/rule-30-%E2%80%93-when-method-class-or)という考え方もある。

どれぐらいが大きくて、どれぐらいがちょうどいいのかは内容による。

とりあえず1ファイルの制限を50行とすると`Change`クラスのファイルが50行を少しだけ超えているけど、現状では使ってないメソッドが2つあるのでこれを削れば50行以内に収まる。

1パッケージはC#では名前空間（とフォルダ分け）ということになる。

お金関係のクラスファイルを「MoneyPackage」というフォルダに全部ぶち込んで、全部名前空間を`OOPPractice.MoneyPackage`に変える。`MoneyPackage`内で違反が出ていなければそのパッケージ内で閉じているということになる。

同様に「StoragePackage」を作る。

最後に`Form1.cs`と`VendingMachine.cs`に

~~~C#
using OOPPractice.MoneyPackage;
using OOPPractice.StoragePackage;
~~~

を付け加えれば良い。

## このエクササイズを終えて

オブジェクト指向を本格的に導入してプログラミングするのは今回が初めてだったので、細かいテクニックも含めて色々勉強になった。

* 手続き型コードがベースにあると、オブジェクト指向でのリファクタリングも考えやすい。（何もない状態でクラス設計するには、OOPの経験が必要）
* Visual Studioの支援機能のおかげで、まだ実装していないメソッドを先に書いたり、コードを大胆に削除しても致命的なエラーが出にくい。（逆に言うと支援機能が不十分だとコーディングスピードが落ちる→PHPなどのスクリプト言語でもVSCodeなどを使って支援機能を充実させる必要がある）
* リファクタリングでは処理の移譲がメイン。
* 新しいクラスの発見によりコードが分散化されていく。

また、9つのルールのうちいくつかは以下のように分類することができるかもしれない。

1. 必要な型の準備

   * すべてのプリミティブ型と文字列型をラップすること
   * ファーストクラスコレクションを使用すること

2. 準備した型への処理の閉じ込め

   * Getter、Setter、プロパティを使用しないこと
   * 1行につきドットは1つまでにすること

3. 新しい型・メソッドの発見

   * （else句を使用しないこと）

   * 1つのクラスにつきインスタンス変数は2つまでにすること

「名前を省略しないこと」は簡単すぎていまいち効果が実感できなかったのと、「一つのメソッドにつきインデントは１段階までにすること 」は出番が無かったのでまだ効果が分からない。

「すべてのエンティティを小さくすること」もファイルを整理しただけで、それぞれのクラスは自然と小さくなっていった。おそらく本当にクラスの行数が増え過ぎたら、新しい型・メソッドの発見につながっていたかもしれない。

「else句を使用しないこと」は記事中にも触れたけど、静的型付け言語ではルールの目的が達成されない可能性がある。三項演算子やswitch構文で書いたとしてもそれは「else句を使わない」に対する屁理屈的な答えであり、本質的なものではない。

## 参考

[C\#\(\.NET\)コレクションの使い分けヒント \- Qiita](https://qiita.com/takutoy/items/cb1e94c36296108e5fd7)
[C\#: Array,List,Collection,Dictionary,HashSet,Queue,Stack,IEnumerable \- Qiita](https://qiita.com/aakasaka/items/fce713a90af011cbd21d)

[キューを利用するには？［C\#／VB］：\.NET TIPS \- ＠IT](https://www.atmarkit.co.jp/ait/articles/1801/31/news023.html)
[スタックを利用するには？［C\#／VB］：\.NET TIPS \- ＠IT](https://www.atmarkit.co.jp/ait/articles/1802/14/news014.html)

[【C\#入門】enum\(列挙型\)の使い方総まとめ\(文字列/int/foreach\) \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/50547)

[ポリモーフィズムを活用するとなぜ if や switch が消えるのか？ \- Qiita](https://qiita.com/Nossa/items/a93024e653ff939115c6)
[\[C\#] 型の一致で処理を分岐をする方法\(型スイッチ, is演算子, switch文\) │ Web備忘録](https://webbibouroku.com/Blog/Article/cs-type-switch)
[第3回　型による分岐の改良 \(2/2\)：特集：C\# 7の新機能詳説 \- ＠IT](https://www.atmarkit.co.jp/ait/articles/1708/02/news013_2.html)