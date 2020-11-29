# 記事表示ループを別ファイルに切り分ける

記事表示ループ

~~~php+HTML
<?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>

<?php endwhile; endif; ?>
~~~

これが`index.php`、`page.php`、`single.php`、`category.php`にそれぞれコピーされてしまっているのでリファクタリングする。

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

## 概要

記事ループと言っても、ループ内のコードは現状2種類ある。

* `index.php`、`category.php`・・・複数記事を並べる用

* `page.php`、`single.php`・・・記事単独を表示する用

ということで、複数記事用の`articles.php`と記事単独用の`article.php`をそれぞれ作り、

* `index.php`、`category.php`・・・`articles.php`をインクルード

* `page.php`、`single.php`・・・`article.php`をインクルード

のようにする。

## 手順

### `articles.php`の作成

`articles.php`というファイルを新規作成し`index.php`または`category.php`から記事ループ部分を丸ごとコピペ。

`articles.php`

~~~php+HTML
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
~~~

### `articles.php`のインクルード

`index.php`と`category.php`から記事ループ部分をバッサリカットして、`<?php get_template_part('articles'); ?>`を挿入。

`index.php`

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <?php get_template_part('articles'); ?> <!-- ←ここ -->
            </div>
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

`category.php`

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <div class="category_title">
                    <p><?php single_cat_title(); ?>カテゴリー　記事一覧</p>
                </div>
                <?php get_template_part('articles'); ?> <!-- ←ここ -->
            </div>
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

えらいスッキリした。これで`artciles.php`をインクルードしてくれる。

### `article.php`の作成とインクルード

同様にする。

`article.php`

~~~php+HTML
<?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
    <div class="border_outer article_outer">
        <div class="border_inner">
            <div class="article">
                <section>
                    <div class="article_header">
                        <h1><?php the_title(); ?></h1>
                    </div>
                    <div class="article_main">
                        <?php the_content(); ?>
                    </div>
                    <div class="article_footer">

                    </div>
                </section>
            </div>
        </div>
    </div>
<?php endwhile; endif; ?>
~~~

`single.php`

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <?php get_template_part('article'); ?> <!-- ←ここ -->
            </div>
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

`page.php`

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <?php get_template_part('article'); ?> <!-- ←ここ -->
            </div>
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

`single.php`と`page.php`が完全に一緒になってしまった。というか元々そうだったのか。

