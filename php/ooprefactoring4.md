# オブジェクト指向まとめ

少ししかしていないけど、もうまとめる。

## オブジェクト指向でプログラミングするなら最初からそういう設計にすべき

今回`Posts`クラスという「DB→View」の流れを担当するクラスを作ったので、「View→DB」の流れを担当するクラスを作ろうとして、結果的に諦めた。

* 1つのファイルからはコードが減ったけど、ファイル数が増えて結果的に行数は増加
* 今までのコードの書き直しが多数発生
* データフローも逆にシンプルではなくなった
* テスト（正確にはデータプロバイダーの書き直し）がかなり面倒

ということで、オブジェクト指向でコードを書きなおすメリットが薄く、デメリットが目立つという状態になってしまった。

今回「とりあえず書こう」というスタンスでコードを書き始めたので、ここからオブジェクト指向を適用しようと思ったら

1. 現状に合わせるために、オブジェクト指向で良いとされているクラスとはかけ離れたねじ曲がったクラスを当てはめる

2. キレイなコードにするために全部最初からやり直す

の2択になりがちで、1だとコードだけ無駄に増えて、しかもやたらと書きなおしが発生し、そしてメリットが薄い。2は、ここまで開発経過を公開してきてさすがに作り直しできないなーと思っただけ。個人で黙々と開発してたら選択していた可能性はある。

ということで、最初からオブジェクト指向を念頭に置いて開発を始めたら良かったね、ということになった。

## もし作り直していたらどうしていたか

詰め始めると別のプログラムができてしまうので、構想だけ。

### 原則

1. 組み込みの関数やオブジェクト（今回の掲示板アプリではPDOオブジェクト）を使うメソッドでは、プリミティブ型（文字列や数値、およびそれらの配列）の変数を使用し手続き型でコーディングする方が良い。
2. 1のようなメソッドを1つまたは複数持つクラスを作成。メソッドの引数もプリミティブ型。また、再利用性を重視。
3. 2のようなクラスをラップするクラスを作成。バリデーションを担当。オブジェクトを受け取りプリミティブ型のデータに直して内部で保存。あるいは
4. 3のラップクラスの外側ではなるべくオブジェクト指向の文脈でコーディングする。

### 実装案

#### DB周り

`DB`クラス（原則の3のレベルに該当）

~~~php
<?php

class DB
{
    // 役割：外部とDBの間にある関所
    //
    // 外部から入ってくるデータを、DBに登録できる形に直す（ステージと呼ぶ）
    // DBから出てきたデータを、外部が受け入れられる形に直す（アンステージと呼ぶ）
    // 
    // PDOオブジェクトはこのクラスの$db内にしかなく、外部から見ればこのクラス自体が
    // DBとして振る舞うように見えるので「DB」という名前にした
    
    private DbSource $db = new DbSource();
    private ?array $stagedData = null;
    private bool $isStaged = false;
        
    public function Stage(Post $post)
    {
        // Postオブジェクトを配列に直す処理を手続き型で書く
        $this->stagedSource = // 配列を代入
        $this->isStaged = true;
    }
    
    public function IsStaged() : bool
    {
        return $this->isStaged;
    }
    
    public function Save()
    {
        if ($this->isStaged)
        {
             $this->db->Create($this->stagedData);
        }
    }
    
    // 他にもGetPosts、GetOnePost、Update、Deleteに当たるメソッドを書く
}
~~~

`DbSource`クラス（原則のレベル2に相当：今の`GetFormAction`クラスとほぼ同じ）

~~~php
<?php
    
class DbSource
{
    private $pdo;
    
    public function __constract()
    {
        try {
            $this->pdo = new PDO(PDO_DSN, DATABASE_USER, DATABASE_PASSWORD);
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
        } catch (PDOException $e) {
            header('Content-Type: text/plain; charset=UTF-8', true, 500);
            exit($e->getMessage());
        }
    }
    
    // 以下、CRUD
    
    public function Create($data)
    {
        // DBに保存する処理
    }
    
    public function ReadAll()
    {
        // DBからほぼすべてのフィールドを読み出す処理
    }
    
    public function Read($data)
    {
        // DBから特定のフィールドを読み出す処理
        
    }
    
    public function Update($data)
    {
        // DBの該当フィールドに対しての更新処理
    }
    
    public function Delete($data)
    {
        // DBの該当フィールドの削除処理
    }
}
~~~

使い方

~~~php
$db = new DB();
$UserInputPost = new Post(); // $_POSTからユーザーが入力したデータを引っ張ってきて何とかしてPostオブジェクトに格納

$db->Stage($UserInputPost);
$db->Save();

$Posts = $db->ReadAll();
~~~

みたいな。

んー・・・

書いてみて思ったけど、やっぱり世間で謳われているようなオブジェクト指向プログラミングのメリットって、小規模な個人開発ではちょっと少ないかも。

逆に大規模もしくは複数人開発ならメリットはあるかな？