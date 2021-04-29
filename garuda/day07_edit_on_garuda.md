# 7日目　このサイトのコンテンツをGaruda Linuxで編集＆アップ自動化

実際このページはGaruda Linux上で書いている。

要は

1. 編集を始める前には必ず`pull`
2. 編集後に`commit`と`push`したらRaspberry Pi上で`git --reset hard`をする。

をすればいいだけ。特に2はWindowsではスクリプトにして自動化しているので、Garuda上でもスクリプトを書けば良い。

## アップ自動化スクリプト

SSH接続を自動化した時と同じ要領でコマンドを自作して、アップするときはそれを実行する。

~~~shell
$ cd ~/command
$ micro syncgit
~~~

以下を記述。

~~~bash
#!/usr/bin/fish

set TARGET_DIRECTORY "/home/takeshi/html"
set COMMIT_MESSAGE (date +%Y/%m/%d-%H:%M:%S)
set REMOTE "upload@192.168.1.201"
set PRIVATE_KEY "/home/takeshi/.ssh/id_rsa_upload"
set PORT 51234
set COMMAND "cd /home/takeshi/www/html && git reset --hard"

cd $TARGET_DIRECTORY
git add .
git commit -m $COMMIT_MESSAGE
git push origin master
ssh -i $PRIVATE_KEY -p $PORT $REMOTE $COMMAND

read -c "Press Enter..." ENTER
~~~

シェルがfishなので文法がbashとは少し違う。当然PowerShellとも違う。でもやってることは一緒。

実行権限を与える。

~~~shell
$ chmod +x syncgit
~~~

いざ実行。

~~~shell
$ syncgit
~~~

