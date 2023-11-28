---
title: 「diffコマンド」と「Gitのdiff」は何が違うのか調査してみた
tags:
  - Linux
  - Git
private: false
updated_at: '2023-09-02T10:28:09+09:00'
id: 6924c419e12cfcaebcba
organization_url_name: null
slide: false
ignorePublish: false
---

以下の2つのテキストファイルがあるとする  

```A.txt
apple
banana
chocolate
```

```B.txt
apple
blueberry
chocolate
```

## A. Linuxのdiffコマンドで比較

```sh
diff A.txt B.txt
```

オプション無しの場合、normal形式のdiffが出力される

```diff
2c2
< banana
---
> blueberry
```

normal形式では、変更行の前後は出力されない

## B. Linuxのdiffコマンドで比較(Gitのdiff風になるようにオプションを付与)

```sh
diff -u A.txt B.txt
```

unified形式のdiffが出力される  
Gitのdiffにかなり近い形式  

```diff
--- A.txt       2022-10-12 17:24:12.165288000 +0900
+++ B.txt       2022-10-12 17:26:09.695288000 +0900
@@ -1,3 +1,3 @@
 apple
-banana
+blueberry
 chocolate
```

ただ、これだと出力文字列に色が付かない...  
diffを色付きで見たい場合、  

```sh
diff -u A.txt B.txt > sabun.diff
```

という風に一旦ファイルに出力し、  
それをVimやVSCodeなどのリッチなエディタで開いてシンタックスハイライトを表示させるという方法があるが、  
「直で色付きで出力させたい！」というせっかちな人には、後述の方法を紹介する  

## C. colordiffコマンドを使用

```sh
sudo apt install colordiff
colordiff -u A.txt B.txt
```

色付きのdiffが出力されて、見やすくなった！  

```diff
--- A.txt       2022-10-12 17:24:12.165288000 +0900
+++ B.txt       2022-10-12 17:26:09.695288000 +0900
@@ -1,3 +1,3 @@
 apple
-banana
+blueberry
 chocolate
```

## D. Gitのdiff

1. 空のディレクトリを作成し、`git init` でGitリポジトリを作成
2. 変更前のファイルをコミット
3. 変更後のファイルに置き換えてコミット

という手順でdiffを作成(面倒...)  
見慣れた形式のdiffが出力される  

```diff
diff --git a/test.txt b/test.txt
index 684da80..6e73a9c 100644
--- a/test.txt
+++ b/test.txt
@@ -1,3 +1,3 @@
 apple
-banana
+blueberry
 chocolate
```

## ところで、Gitはどうやって差分を取得している？

```sh
# テキストファイルのハッシュ値を計算
cat test.txt | git hash-object -w --stdin
```

> 6e73a9cea3235240ff29f184e0427d47b09f8801

diffを表示した際に出力された `index 684da80..6e73a9c` の `6e73a9c` は、ファイルの中身をハッシュ化した際の値の先頭7桁と一致！
Gitはハッシュ関数を利用することで、ファイルの中身が一致しているかどうかをチェックしているというわけなのだ  

ちなみに、ハッシュ値の先頭2桁はディレクトリ名、残りの38桁はファイル名として `.git/objects` の下に保存される  
![git_object.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/675511/d0124213-4c14-d2fb-0ad4-b3449b121dbc.png)

`cat-file` コマンドを使うことで中身を取り出すこともできる  

```sh
git cat-file -p 6e73a9cea3235240ff29f184e0427d47b09f8801
```

> apple  
> blueberry  
> chocolate  

Gitが高速な動作をする背景には、こういった優れた仕組みがあったのだ  
リーナス様に感謝🙇‍♂️

## 参考サイト

- [10.2 Gitの内側 - Gitオブジェクト](https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-Git%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)
- [git diff コマンドとふつうの diff / patch コマンド](https://gotohayato.com/content/108/)
- [【 diff 】コマンド（基本編）――テキストファイルの差分を出力する](https://atmarkit.itmedia.co.jp/ait/articles/1704/13/news021.html)
- [git diff | Atlassian Git Tutorial](https://www.atlassian.com/ja/git/tutorials/saving-changes/git-diff)
- [Gitを支える内部構造についての話](https://techblog.timers-inc.com/entry/2016/11/14/113154)
- [Gitのコミットハッシュ値は何を元にどうやって生成されているのか](https://engineering.mercari.com/blog/entry/2016-02-08-173000/)
