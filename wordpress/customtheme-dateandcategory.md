# 記事に「投稿日付」と「所属カテゴリタグ」を付ける

記事ループのリファクタリングを終えたところで、記事ループ内の調整をしていく。

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

## 投稿日付

`echo get_the_date();`で取得できるらしい。

`articles.php`

~~~php+HTML
～略～
<section>
    <div class="article_summary_header">
        <a href="<?php echo the_permalink( $post ); ?>"><h1><?php the_title(); ?></h1></a>
        <p><time><?php echo get_the_date(); ?></time></p> <!-- ←ここ -->
    </div>
    <div class="article_summary_main">
        <?php the_content(); ?>
    </div>
    <div class="article_summary_footer">
        <p><?php comments_number(); ?></p>
        <p>カテゴリ</p>
    </div>
</section>
～略～
~~~

`article.php`

~~~php+HTML
～略～
<section>
    <div class="article_header">
        <h1><?php the_title(); ?></h1>
        <p><time><?php echo get_the_date(); ?></time></p> <!-- ←ここ -->
    </div>
    <div class="article_main">
        <?php the_content(); ?>
    </div>
    <div class="article_footer">

    </div>
</section>
～略～
~~~

日付表示のフォーマットをWordPressの管理画面から選ぶ。

![image-20201129154441125](image/customtheme-dateandcategory/rs-image-20201129154441125.png)

良い。

![image-20201129154640050](image/customtheme-dateandcategory/rs-image-20201129154640050.png)

## 記事の所属カテゴリー

`the_category()`で表示されるらしい。

`articles.php`

~~~php+HTML
<section>
    <div class="article_summary_header">
        <a href="<?php echo the_permalink( $post ); ?>"><h1><?php the_title(); ?></h1></a>
        <p><time><?php echo get_the_date(); ?></time></p>
    </div>
    <div class="article_summary_main">
        <?php the_content(); ?>
    </div>
    <div class="article_summary_footer">
        <p><?php comments_number(); ?></p>
        <p><?php the_category(); ?></p> <!-- ←ここ -->
    </div>
</section>
~~~

`article.php`

~~~php+HTML
<section>
    <div class="article_header">
        <h1><?php the_title(); ?></h1>
        <p><time><?php echo get_the_date(); ?></time></p>
    </div>
    <div class="article_main">
        <?php the_content(); ?>
    </div>
    <div class="article_footer">
        <p><?php the_category(); ?></p> <!-- ←ここ -->
    </div>
</section>
~~~

リンクの色が気に入らないけど、まあこんなもんか。

![image-20201129160154772](image/customtheme-dateandcategory/rs-image-20201129160154772.png)

## 参考

[テンプレートタグ/get the date \- WordPress Codex 日本語版](https://wpdocs.osdn.jp/%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%BF%E3%82%B0/get_the_date)
[テンプレートタグ/the category \- WordPress Codex 日本語版](https://wpdocs.osdn.jp/%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%BF%E3%82%B0/the_category)