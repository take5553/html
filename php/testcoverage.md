# テストの罠

[前回](testanddebug.html)でテストが成功しこれで大丈夫かと思いきや、実はまだ穴があるという話。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - XDebug 2.8.1
  - PHPUnit 9.0.0

## テストを変更

`testtestAdd`を以下のように変更。

~~~php
    public function testtestAdd()
    {
        $testtest = new TestTest;
        $this->assertEquals(1, $testtest->testAdd(2, -1));
    }
~~~

`2 + (-1) = 1`でしょ。

ところが、テストをしてみると失敗する。

~~~
> ./phpunit tests/
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

.F                                                                  2 / 2 (100%)

Time: 00:26.167, Memory: 4.00 MB

There was 1 failure:

1) TestTestTest::testtestAdd
Failed asserting that 2 matches expected 1.

C:\work\html\raspberrypi-server\test\html\php-bbs\tests\TestTestTest.php:18

FAILURES!
Tests: 2, Assertions: 2, Failures: 1.
~~~

まあ足し算と言いながら引き算しているので失敗するのは当然なのかもしれないけど、`testAdd`の引数は`int`型なので負の数も許してしまう。つまり`testAdd`は不具合を起こす可能性を含んでいる。

でも[前回](testanddebug.html)のテストでは`100%`と出てすべてのテストは成功している。

ということは、前回のテストは不具合の可能性を見つけられないテストだった、ということになる。

## 良いテストとは

何かの処理に対するテストとして「どんなテストを行うか」というのはとても大事。でも良いテストはどうやったら書けるのかというのは難しい問題。

自分もテストを書いた経験があまりないので何とも言えないけど、とりあえずのとっかかりとして

* 正常系・準正常系・異常系
* 同値分割・境界値分析

というものがあるらしい。



## 参考

[初心者エンジニアがたどり着いたテスト仕様書の書き方 \- Qiita](https://qiita.com/nnahito/items/a664d89ff8a6c7a741b2)
[「正常系」・「準正常系」・「異常系」テストについて、まとめてみました。 \- ヒャッハー！ふっくら太郎です。](http://gokushiteki-softtest.hatenablog.com/entry/2017/09/09/170828)
[失敗しないテストケースの作り方と、効率よくテストを進める方法 \| クラウド型テスト管理ツール「Qangaroo（カンガルー）」](https://qangaroo.jp/info/test-case-plan-do/)
[テストケースの作成 \- Qiita](https://qiita.com/salvage0707/items/b15412af6951da2069ba)