# 404ページを作る

現状では適当に`http://(WordPressのURL)/asdf`とか打って存在しない記事にアクセスしようとすると、

![image-20201129172941420](image/customtheme-404/rs-image-20201129172941420.png)

これはひどい。

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

## 手順

`404.php`という名前でファイルを新規作成し、`index.php`の内容を丸ごとコピペ。

`404.php`

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <div class="container articles">
                <?php get_template_part('articles'); ?>
            </div>
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

そして、`<?php get_template_part('articles'); ?>`の部分を削除し、ページが見つからない旨のメッセージを書けば良い。

`404.php`（変更後）

~~~php+HTML
<?php get_header(); ?>

    <main>
        <div class="wrapper">
            <!-- ----------ここから---------- -->
            <div class="container noarticles">
                <h2>404 Not Found</h2>
                <p>お探しのページは見つかりませんでした。</p>
                <img src="<?php echo get_template_directory_uri(); ?>/image/rs-pose_syazai_man.png" alt="404">
                <a href="<?php echo esc_url( home_url() ); ?>">トップページ</a>
            </div>
            <!-- ----------ここまで---------- -->
            <div class="container sidebar">
                <?php get_sidebar(); ?>
            </div>
        </div>
    </main>

<?php get_footer(); ?>
~~~

`div`タグに`noarticles`クラスを付与し、再スタイリングした。

使用画像。

![謝罪をする人のイラスト（男性）](image/customtheme-404/pose_syazai_man.png)

こんな感じ。

![image-20201129175700405](image/customtheme-404/rs-image-20201129175700405.png)

まあいいんじゃないか。