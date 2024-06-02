---
title: "Gitコマンドメモ_Gitコマンド100本ノックの記事より"
pubDate: 2023-01-03T11:24:10+09:00
description: 'useful git command'
draft: false
---

Twitterで以下の記事を目にしました。
[意外と知らない？ Gitコマンド 100本ノック](https://qiita.com/ueki05/items/5c233773e3186989bfd3)

自分の使用頻度が少ない&便利なコマンドの、自分用メモです


## コマンド
`git config --global user.email "your-email@example.com"`
- 設定

`git config user.name`
- 設定値を表示

`git log --stat`
- 変更内容をファイルとともに表示

`git add --dry-run .`
- git addで何か実行されるか示す

`git log --shortstat --oneline`
`git log --stat --oneline`
- 1行で表示
- ログが簡略化されてみやすくなる

`git log --parents`
- 親コミットのSHA1 IDを表示
- SHA1 IDを短縮表示する場合

`git log --oneline`
- 簡潔表示

`git log --patch`
- ファイルの差分を表示する

`git log --patch-with-stat`
- ファイルの差分と、stat(更新対象のファイル一覧表示、など)を同時に表示する

`git log --graph --decorate --pretty=oneline --all --abbrev-commit`
- コミット履歴を全てのブランチを通じて閲覧

`git log --graph`
- コミット履歴をブランチの推移と合わせてグラフィカルに表示する

`git config --get-regexp BRANCH`
- 名前にBRANCHという言葉が含まれているGit構成設定のリストを表示する

`git push origin :REMOTE_BRANCH`
- REMOTE_BRANCHという名前のbranchを、リモートから削除する

`git log --merges`
- マージの結果であるコミットのリストを表示

`git log FILE`
- FILEに影響を与えたコミットのリストを表示
- git logのオプションと組み合わせるとさらに調べやすくなる
- e.g.
	- `git log --patch-with-stat FILE`

`git log --grep=STRING`
- git logにSTRINGの文字列を含むものを表示
- commitのprefixを指定すると便利だと思う

`git log --since MM/DD/YYYY --until MM/DD/YYYY`
- 二つの日付の間に作成されたcommitの表示

`git shorlog`
- commiterによる、commitの要約
- `-e`
  - メールアドレスも表示する

`git grep STRING`
- STRINGを含む全てのファイルを探し出す
- git 管理されているレポジトリでは、こちらのコマンドは便利だと思った

# 感想
- gitのコマンドのオプションの理解を含めると、オプションを組み合わせて目当ての情報を表示できると思った
  - オプションを含めてフレーズとして覚えてしまうのではなく、各オプションの意味を理解するイメージ
- どこかのタイミングでgitもまとめて勉強してみたい
  

