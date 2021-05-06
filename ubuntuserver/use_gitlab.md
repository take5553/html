# GitLabにtakeshiユーザーを追加

一旦rootユーザーからサインアウトして、ログイン画面に戻り、画面下部の`Register now`をクリック。

![image-20210506232804315](image/use_gitlab/image-20210506232804315.png)

適当に登録するとログイン画面に戻るので、ユーザー名とパスワードを入れてログイン！

![image-20210506233005982](image/use_gitlab/image-20210506233005982.png)

あら、管理者の承認がいるらしい。

再びrootでログインし、画面上部の「More」から「Admin area」に入る。

![image-20210506233232143](image/use_gitlab/image-20210506233232143.png)

「View latest users」をクリック。

![image-20210506233259167](image/use_gitlab/image-20210506233259167.png)

「Pending approval」の中にいた。歯車マークに「Approve」があるので認証してあげる。

![image-20210506233508698](image/use_gitlab/image-20210506233508698.png)

ではrootはサインアウトして、takeshiユーザーで再度ログイン。

![image-20210506233805451](image/use_gitlab/image-20210506233805451.png)

ええー。Firefoxに止められた。適当にしかドメイン登録してないからか。

その後ブラウザを変えたりクッキー削除したりしてみたけどダメだったので、管理者の認証を無しにする。

「Admin area」の画面左の一番下にある歯車マークから、「General」をクリック。

![image-20210506235509078](image/use_gitlab/image-20210506235509078.png)

「Sign-up restrictions」にある「Require admin approval for new sign-ups」のチェックを外す。

![image-20210506235117655](image/use_gitlab/image-20210506235117655.png)

設定を保存。

![image-20210506235134127](image/use_gitlab/image-20210506235134127.png)

takeshiユーザーを一旦削除（Approveしたときの画面からできる）して、再度登録。

Welcome画面を撮り忘れたけど、無事入れた。

![image-20210506235825386](image/use_gitlab/image-20210506235825386.png)

