---
title: "Usage of Df and Du"
description: 'Usage of Df and Du'
pubDate: 2023-01-03T10:00:00+09:00
draft: false
---


# df/duコマンド
- du/dfコマンドを整理する
- ※注意
    - 内容の正確性について保証しません。osやコマンドの世代によって実行方法に違いが出る可能性があるため、実行時の環境のマニュアルを参考にしてください
## dfコマンド
- 使用中のディスクの使用量を確認する
- ディスクの使用量を理解しやすくするために、`-h`オプションがよく使用される
- https://beyondjapan.com/blog/2017/07/du-comands_df-comands/
### 2進数ベースか、10進数ベースか
- より正確に表すために、2進数ベースか、10進数ベースか区別するようになった
- 詳細は以下の記事参照
    - https://wa3.i-3-i.info/diff20tani.html
    - https://blog.maromaro.co.jp/archives/8900
    - https://academy.gmocloud.com/keywords/20170510/4314
- macの場合、以下の通りに違いが出る

```bash
# 10進数ベースの場合。一般的に用いられる値
$ df -H
Filesystem       Size   Used  Avail Capacity iused      ifree %iused  Mounted on
/dev/disk3s1s1   494G   8.9G   360G     3%  348574 3515551600    0%   /
devfs            209k   209k     0B   100%     708          0  100%   /dev
/dev/disk3s6     494G   2.1G   360G     1%       2 3515551600    0%   /System/Volumes/VM

# 省略

# 2新数ベースの場合。コンピュータで用いられるのは2進数だから、10進数の場合より、実態としての値に近い
$ df -h
Filesystem       Size   Used  Avail Capacity iused      ifree %iused  Mounted on
/dev/disk3s1s1  460Gi  8.3Gi  335Gi     3%  348574 3515551560    0%   /
devfs           205Ki  205Ki    0Bi   100%     708          0  100%   /dev
/dev/disk3s6    460Gi  2.0Gi  335Gi     1%       2 3515551560    0%   /System/Volumes/VM

# 省略

```

## duコマンド

- コマンドを実行した階層以下のフォルダ容量を表示する
	- `-h`
		- 理解しやすい表示
		- `“Human-readable” output.  Use unit suffixes: Byte, Kilobyte, Megabyte, Gigabyte, Terabyte and Petabyte based on powers of 1024.`
### duコマンドの実践方法

  - デフォルトの挙動
	  - カレントディレクトリ以下のファイルフォルダ容量を全て調べてしまう
	  - 以下の例だと、`typescript`と同じ階層のディレクトリの容量を調べようとしても、それ以下の階層が全て対象となってしまう

```bash
$ du -h
# 省略
...
4.0K	./typescript/typescript-react/react-sample/node_modules/workbox-build/node_modules/....
# 省略

```

- 調査対象の階層を指定する
    - オプションで階層を指定することでスキャンの深さを指定できる
        - linuxの場合
            - `--max-depth=1,2,...`
        - macの場合
            - `-d`
              - https://shinya131-note.hatenablog.jp/entry/2015/08/06/095911
              - https://ma.ttias.be/du-max-depth-alternative-mac-osx/
    - カレントディレクトリを変更せずとも、引数で調査対象のディレクトリを指定可能
- ソート
    - `sort`コマンドと組み合わせると、容量順に表示可能
- 除外
    - `--exclude=` linuxの場合
- https://beyondjapan.com/blog/2017/07/du-comands_df-comands/

```bash
$ du -h -d 1
4.0K	./go
  0B	./python
 20K	./shell
1.2G	./typescript
356M	./rust

#省略

$ du -h -d 2
4.0K	./go/go-calculate
4.0K	./go
  0B	./python
 20K	./shell/shell-training
 20K	./shell
379M	./typescript/typescript-react
801M	./typescript/survival-typescript
1.2G	./typescript
356M	./rust/sample-web-app
356M	./rust

#省略

$ du -h -d 1 ./typescript
379M	./typescript/typescript-react
801M	./typescript/survival-typescript
1.2G	./typescript

$ du -h -d 1 | sort -h -r
3.3G	.
1.2G	./typescript
720M	./react
554M	./nextjs
438M	./flutter
356M	./rust
171M	./sql
```

## この後やること
- 以下の単語がいまいちわかっていないから、調べる
    - ファイルシステム
    - パーテーション
