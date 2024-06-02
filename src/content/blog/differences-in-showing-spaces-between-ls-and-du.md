---
title: "lsとduを用いて確認する容量の表示の違い--ファイルの論理的なサイズ表示とブロック単位の表示"
description: "difference between ls and du command"
pubDate: "2023-01-07T13:05:00+09:00"
---
気になったので調べてみた

lsコマンドと、duコマンドで表示されるディレクトリサイズが違っていた


以下の記事がほぼ解説してくれていた。
https://www.unknownengineer.net/entry/2019/01/08/171500

参考にしつつ、記事内で調べた内容を自分の環境でも実演する

# 容量の表示の違い--ファイルの論理的なサイズ表示と、ブロック単位の表示
 
- コンピュータの消費ディスク量をチェックするときに、ファイルのサイズと、ブロックサイズを見分ける必要がある
- 新しく学んだことだからメモする


## 疑問に思っていたこと
ディスク容量を表示する`du` と、`ls`で表示の違いが気になっていた

## コンピュータ上でのディスクの使い方の考え方

du コマンドとlsコマンドの表示容量の違いから調べていたら、以下の記事に行き着いた

実際のファイルサイズと、ディスクの消費量は異なる
- `ls -l`はファイルの論理的な意味のバイト数を表示する
- `du` は、ブロック単位でファイルサイズを表示する
    - `du`は*disk usage*の略称
        - https://www-creators.com/archives/5744
- すなわち、一般的には `論理的な意味のサイズ < ディスクのファイル`となる
    - 引数無しで実行した場合には、**lsで表示されるサイズ < duで表示されるサイズ**
- オプションを指定することで、ファイルのバイト数と、ブロック単位のバイト数と、表示を分けることができる
  - https://okwave.jp/qa/q4032965.html
  - https://hydrocul.github.io/wiki/commands/du.html

実際に試してみる
※いずれもmacの場合
中身が一文字のファイルの容量を調べる
結果は以下
- `ls`は実際のファイル容量を表示している
- duは確保されるブロック単位でのディスク容量を指定している

```sh
~/t/file-size ❯❯❯ cat file.txt
a
~/t/file-size ❯❯❯ ls -lh file.txt
-rw-r--r--  1 user  staff     2B Jan  7 14:37 file.txt
~/t/file-size ❯❯❯ du -h file.txt
4.0K	file.txt
~/t/file-size ❯❯❯
```

macの場合はブロック単位は4096Byte
ファイルサイズが、2Byteでも、1ブロック(4096Byte≒4.0K)が確保されることがわかる
```sh
~/t/file-size ❯❯❯ stat -f %k .
4096
~/t/file-size ❯❯❯

```

では、サイズが4096Byte以上のファイルを用意する
`a + 改行`でちょうど2Byte増加する。

```sh
~/t/file-size ❯❯❯ cat file.txt
a
a
a
~/t/file-size ❯❯❯ ls -lh file.txt
-rw-r--r--  1 staff  staff     6B Jan  7 14:56 file.txt
~/t/file-size ❯❯❯ echo a >> file.txt
~/t/file-size ❯❯❯ cat file.txt
a
a
a
a
~/t/file-size ❯❯❯ ls -lh file.txt
-rw-r--r--  1 staff  staff     8B Jan  7 14:57 file.txt
~/t/file-size ❯❯❯


```

`a + 改行`を、2048回書き込んだファイルを準備する

```sh
~/t/file-size ❯❯❯ cat output.sh
#!/bin/sh

for i in {0..2047}; do
	echo "a" >> check-file-size.txt;
done

~/t/file-size ❯❯❯
~/t/file-size ❯❯❯ chmod +x output.sh
~/t/file-size ❯❯❯ ./output.sh
~/t/file-size ❯❯❯ wc -l check-file-size.txt
    2048 check-file-size.txt
# 2048行書き込まれている

```

ファイルサイズを確認
- ちょうど4096バイト
- `du`で確認すると、作成した二つのファイルともに、確保している容量は同じ
```sh
~/t/file-size ❯❯❯ ls -l
total 24
-rw-r--r--  1 user  staff  4096 Jan  7 15:07 check-file-size.txt
-rw-r--r--  1 user  staff     8 Jan  7 14:57 file.txt
-rwxr-xr-x  1 user  staff    74 Jan  7 15:07 output.sh

~/t/file-size ❯❯❯ du -d 1 *f*
4.0K	check-file-size.txt
4.0K	file.txt
~/t/file-size ❯❯❯

```

では、macの1blockのサイズ4096Byteより、ファイルを大きくする

```sh
~/t/file-size ❯❯❯ cat check-file-size.txt
# 中略
a
a
~/t/file-size ❯❯❯ 

# 改行を一つ挿入する
~/t/file-size ❯❯❯ echo -e "" >> check-file-size.txt
~/t/file-size ❯❯❯ cat check-file-size.txt
# 中略
a
a

~/t/file-size ❯❯❯ cat check-file-size.txt
```

ファイルサイズを確認
1Byteのみ大きくなっている
```sh

# ファイルサイズを確認
~/t/file-size ❯❯❯ ls -l
total 32
-rw-r--r--  1 user  staff  4097 Jan  7 15:21 check-file-size.txt
-rw-r--r--  1 user  staff     8 Jan  7 14:57 file.txt
-rwxr-xr-x  1 user  staff    74 Jan  7 15:07 output.sh
```

duコマンドで確認すると、8KByte分確保されている！

```sh

# duコマンドでdiskの使用量を確認
~/t/file-size ❯❯❯ du -d 1 *f*
8.0K	check-file-size.txt
4.0K	file.txt
~/t/file-size ❯❯❯

```

## まとめ
- ファイルの論理的なサイズと、ディスクのブロック単位での確保の違いを理解できた

## 次に調べること
- コンピュータのディスクの保存の仕組み

### わからなかったこと

- `du -A` は何を示しているのか?

```sh
~/t/file-size ❯❯❯ ls -l
total 32
-rw-r--r--  1 user  staff  4097 Jan  7 15:21 check-file-size.txt
-rw-r--r--  1 user  staff     8 Jan  7 14:57 file.txt
-rwxr-xr-x  1 user  staff    74 Jan  7 15:07 output.sh

~/t/file-size ❯❯❯ du *
8.0K	check-file-size.txt
4.0K	file.txt
4.0K	output.sh

~/t/file-size ❯❯❯ du -A *
4.5K	check-file-size.txt
512B	file.txt
512B	output.sh

```

`man du` を参照すると、`見た目のサイズを表示する`をあったから、`ls -l` の結果と一致すると思ったが、数値が異なっている

```sh
NAME
     du – display disk usage statistics
# 中略
    The options are as follows:

     -A      Display the apparent size instead of the disk usage.  This can be helpful when operating on compressed volumes or sparse files.
     
# 中略
```


- `ls -s`コマンドのブロックサイズは何を基準としているのか？
    - ブロックサイズは、4096Byteなので、`check-file-size.txt`は、2と思ったら、16となっている

```sh
~/t/file-size ❯❯❯ ls -sl
total 32
16 -rw-r--r--  1 user  staff  4097 Jan  7 15:21 check-file-size.txt
 8 -rw-r--r--  1 user  staff     8 Jan  7 14:57 file.txt
 8 -rwxr-xr-x  1 user  staff    74 Jan  7 15:07 output.sh

```
