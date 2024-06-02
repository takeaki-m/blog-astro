---
title: "gitconfigファイルをGitHub管理にしコンピュータ間で共有する"
description: "gitconfigファイルをGitHub管理にしコンピュータ間で共有する"
pubDate: 2023-07-03T22:30:17+09:00
---

ずっとやりたかったが、一念発起して実行した。
おかげて複数のコンピュータ間でgitコマンドが共通して使えるようになりすごく楽になった。

zshなど他の.ファイル系共有方法は同じだが、.gitconfigファイルのみ共有方法に躓いたので備忘を兼ねてメモしておく。


## 一般的なファイルdotfileの共有方法--シンボリックリンクを貼る
ググるといくらでも出てくる方法だが、一般的なdotfileの共有方法は、シンボリックリンクを貼るスタイルである。

ざっくりいうと、
- dotfileを保存するレポジトリを作成
- レポジトリにて各種のdotfileを保存
- 各コンピュータは、上記のレポジトリをclone
- 各種設定ファイルは、レポジトリのファイルに対して、シンボリックリンクを貼る

こんな流れで設定する

## 直面した問題--.gitconfigが読み込まれない

これと同様のスタイルで、.gitconfigも設定しようと思ったのだが、なぜか「シンボリックリンクは貼られているが、.gitconfigが読み込まれない」という状況に落ちいてしまった。
.gitconfigはただのファイル(zshなどと違い、再起動や読み込みは不要の認識)なのに、読み込まれなかった


## 問題に対する解決策
ただのファイルだから再読み込みもないだろうと思い他の方の設定を調べていたら、以下の方の設定が参考になり拝借した

https://github.com/kenkenpa198/dotfiles#gitignore_shared-と-gitignore_global


具体的に参考になったのは以下の通り
> .gitconfig を直接管理するのではなく .gitconfig_shared に切り分けて外部読み込みを行う運用にしている。.gitconfig には Git のユーザー名とメールアドレスを記述する必要があり、Git 管理の対象にしたくなかったため。


.gitconfigを直接管理するのではなく、切り分けて外部読み込みしている点
特にuser name と emailを外出してできる。
利用するコンピュータによって、user name や emailは異なっているからこれはすごく役立った(email以外にも、各環境ごとに異なる設定はこの方法で管理できる)

一応晒しておくと、自分の設定は以下の通りです
https://github.com/takeaki-m/dotfiles/tree/develop/gitconfig



shellファイルを作成して、新しいPCでも一発で共通のgitconfig設定になるようにしました
https://github.com/takeaki-m/dotfiles/blob/develop/gitconfig/git_setup.sh


次はzsh系の管理方法も書きたい




