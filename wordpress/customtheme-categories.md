# カテゴリー一覧ページを作る

カテゴリーごとに分類された記事のみ表示するようなページ。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2
  - PHP 7.3.19-1~deb10u1
  - WordPress 5.5.3-ja

## 準備1

そもそもカテゴリー分けするほど記事を書いていないので、カテゴリーを作り適当に記事を書く。

料理カテゴリ。

![image-20201123214643982](image/customtheme-categories/rs-image-20201123214643982.png)

トンテキ。

![image-20201123232046615](image/customtheme-categories/rs-image-20201123232046615.png)

ぶり大根。

![image-20201123232145974](image/customtheme-categories/rs-image-20201123232145974.png)

## 気付いた不備

* `index.php`に表示される記事の日付が変わっていない
* 個別記事ページに日付が無い
* コメント記入欄が無い（個別記事テンプレートを作るときに必要だった）
* カテゴリー「料理」へのリンクがどこにもない
* 記事のカテゴリー表示が全くない

## 作業概要

* カテゴリー一覧テンプレートを`index.php`から流用する
* 各カテゴリーへのリンクを張る

記事日付と所属カテゴリー表示、コメント欄作成は別記事にまとめる。

## 準備2

テスト環境にも適当にカテゴリー記事を追加。

![image-20201129090343039](image/customtheme-categories/rs-image-20201129090343039.png)

## 手順

### カテゴリー一覧テンプレートを`index.php`から流用する

カテゴリーページは`category.php`という名前で保存すればいいらしい。

`category.php`（中身は`index.php`の丸写し）

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
                    <div class="border_outer article_outer">
                        <div class="border_inner">
                            <div class="article_summary">
                                <div class="thumbnail">
                                    <?php
                                        if ( has_post_thumbnail()) {
                                            the_post_thumbnail( 'thumbnail' );
                                        } else {
                                            echo '<img src=' . get_template_directory_uri() . '/image/rs-no_image_yoko.jpg" alt="No Image">';
                                        }
                                    ?>
                                </div>
                                <section>
                                    <div class="article_summary_header">
                                        <a href="<?php echo the_permalink( $post ); ?>"><h1><?php the_title(); ?></h1></a>
                                        <p><time>2020/10/21</time></p>
                                    </div>
                                    <div class="article_summary_main">
                                        <?php the_content(); ?>
                                    </div>
                                    <div class="article_summary_footer">
                                        <p><?php comments_number(); ?></p>
                                        <p>カテゴリ</p>
                                    </div>
                                </section>
                            </div>
                        </div>
                    </div>
                <?php endwhile; endif; ?>
            </div>
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

ただ、全く一緒だとよく分からないので、「料理カテゴリー　記事一覧」みたいな文字列は表示したい。

現在のカテゴリー名を表示するWordPress関数は

~~~php
single_cat_title();
~~~

だそうだ。

このようにして使う。

~~~php+HTML
<p><?php single_cat_title(); ?>カテゴリー　記事一覧</p>
~~~

これをループの外に書く。ついでに`div`タグで囲んでおく。

~~~PHP+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <!-- ――――――――――ここから―――――――――― -->
                <div class="category_title">
                    <p><?php single_cat_title(); ?>カテゴリー　記事一覧</p>
                </div>
                <!-- ――――――――――ここまで―――――――――― -->
                <?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
                    <div class="border_outer article_outer">
                        ～略～
~~~

とりあえず表示させてみる。`http://(WordPressのURL)/category/cooking`でアクセスできる。

![image-20201129092904504](image/customtheme-categories/rs-image-20201129092904504.png)

まあ、装飾何にもしてないからな。ということでちょっと見やすくする。

`mystyle.css`に追記。

~~~css
/* カテゴリーページ用 */
.category_title {
    font-size: 30px;
    font-weight: bold;
    margin: 20px 0;
}
~~~

![image-20201129093348643](image/customtheme-categories/rs-image-20201129093348643.png)

![image-20201129093542101](image/customtheme-categories/rs-image-20201129093542101.png)

まあこんなもんか。

### 各カテゴリーへのリンクを張る

`wp_list_categories()`で、リンク付きのカテゴリーが出力できるらしい。`header.php`の、カテゴリーリンクの予定にしていたところにPHPコードを貼り付ける。

`header.php`

~~~php+HTML
<header>
    <div class="wrapper">
        <div class="container site-title">
            <a href="<?php echo esc_url( home_url() ); ?>" alt="Blog Link">MY <span>B</span>log</a>
        </div>
        <div class="container nav">
            <nav>
                <ul>
                    <li><a href="<?php echo esc_url( get_page_link( 22 ) ); ?>">このブログについて</a></li>
                    <?php wp_list_categories(); ?> <!-- ←ここ -->
                </ul>
            </nav>
        </div>
    </div>
</header>
~~~

ありゃ、なんか違う。

![image-20201129104413867](image/customtheme-categories/rs-image-20201129104413867.png)

これの解決は`wp_list_categories('title_li=')`という感じで引数`title_li`に何も渡さなければいいらしい。

修正したのがこちら。いい感じ。

![image-20201129104958535](image/customtheme-categories/rs-image-20201129104958535.png)

`wp_list_categories('title_li=&exclude=1')`としてやればUncategorizedだけ非表示にできる。

![image-20201129105350751](image/customtheme-categories/rs-image-20201129105350751.png)

文字列を表示するだけではなく、ちゃんとリンクになっているところが良い。

Raspberry Piにアップロードして確認。

![image-20201129111613646](image/customtheme-categories/rs-image-20201129111613646.png)

英語カテゴリーへのリンクが表示されていないけど、記事が一つも無いから表示しないっぽい。賢い。

## 参考

[テンプレートタグ/single cat title \- WordPress Codex 日本語版](http://wpdocs.osdn.jp/%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%BF%E3%82%B0/single_cat_title)
[テンプレートタグ/wp list categories \- WordPress Codex 日本語版](http://wpdocs.osdn.jp/%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%BF%E3%82%B0/wp_list_categories)