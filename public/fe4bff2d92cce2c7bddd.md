---
title: 最小限のロガー(50行)を自作
tags:
  - C
  - C++
private: false
updated_at: '2022-12-14T11:58:24+09:00'
id: fe4bff2d92cce2c7bddd
organization_url_name: null
slide: false
---

## 実装した機能

- コンパイル時にログレベルを指定し、それ以上のログのみをprintfで出力する機能

## 無い機能

- 出力をファイルに保存する機能
- 時刻を出力する機能

## 作った経緯

- ログレベルを指定することで不要な出力をカットできるようにしたい
- 最速のロガーが欲しい
- 何らかの事情でOSSのロガーライブラリは使えない

というシチュエーションでお手軽に使えるロガーが欲しくて自作した、という経緯でした  
printfしか使っておらず、余計な物が一切存在しないので、多分これが一番早いと思います  
(これをロガーと呼べるのかどうかは微妙ですが...)

## 実装

https://github.com/YossyHal/simple-c-logger

```cpp:logger.h
#pragma once
#include <stdio.h>

#define LOG_LEVEL_TRACE 0
#define LOG_LEVEL_DEBUG 1
#define LOG_LEVEL_INFO 2
#define LOG_LEVEL_WARN 3
#define LOG_LEVEL_ERROR 4
#define LOG_LEVEL_CRITICAL 5
#define LOG_LEVEL_OFF 6

#ifndef LOG_LEVEL
#define LOG_LEVEL LOG_LEVEL_INFO
#endif

#if LOG_LEVEL <= LOG_LEVEL_TRACE
#define LOG_TRACE(...) printf(__VA_ARGS__)
#else
#define LOG_TRACE(...) (void)0
#endif

#if LOG_LEVEL <= LOG_LEVEL_DEBUG
#define LOG_DEBUG(...) printf(__VA_ARGS__)
#else
#define LOG_DEBUG(...) (void)0
#endif

#if LOG_LEVEL <= LOG_LEVEL_INFO
#define LOG_INFO(...) printf(__VA_ARGS__)
#else
#define LOG_INFO(...) (void)0
#endif

#if LOG_LEVEL <= LOG_LEVEL_WARN
#define LOG_WARN(...) printf(__VA_ARGS__)
#else
#define LOG_WARN(...) (void)0
#endif

#if LOG_LEVEL <= LOG_LEVEL_ERROR
#define LOG_ERROR(...) fprintf(stderr, __VA_ARGS__)
#else
#define LOG_ERROR(...) (void)0
#endif

#if LOG_LEVEL <= LOG_LEVEL_CRITICAL
#define LOG_CRITICAL(...) fprintf(stderr, __VA_ARGS__)
#else
#define LOG_CRITICAL(...) (void)0
#endif

```

## 使用例(C言語)

```cpp:hello.c
#include "logger.h"

int main()
{
    LOG_TRACE("trace\n");
    LOG_DEBUG("debug\n");
    LOG_INFO("info\n");
    LOG_WARN("warn\n");
    LOG_ERROR("error\n");
    LOG_CRITICAL("critical\n");
}
```

デフォルトでは INFO 以上のログを出力

```log
$ gcc hello.c && ./a.out
info
warn
error
critica
```

コンパイル時に LOG_LEVEL を指定

```log
$ gcc hello.c -DLOG_LEVEL=LOG_LEVEL_TRACE && ./a.out
trace
debug
info
warn
error
critical
```

LOG_LEVEL_TRACE を指定したため、TRACE以上のログが出力された

## 使用例(C++)

```cpp:hello.cpp
#include "logger.h"

int main()
{
    LOG_TRACE("trace\n");
    LOG_DEBUG("debug\n");
    LOG_INFO("info\n");
    LOG_WARN("warn\n");
    LOG_ERROR("error\n");
    LOG_CRITICAL("critical\n");
}
```

使い方はC言語と同じ

```log
$ g++ hello.cpp -DLOG_LEVEL=LOG_LEVEL_TRACE && ./a.out
trace
debug
info
warn
error
critical
```

## ちなみに

ログレベルより低いログはコンパイル時にコードごと削除されるので、  
ログをいっぱい書いても処理速度が落ちたりする心配はありません  

```cpp:test.cpp
#include "logger.h"

int main()
{
    int i = 10;
    LOG_TRACE("i = %d", i);
    return 0;
}
```

```log
$ g++ test.cpp -Wall
test.cpp: In function ‘int main()’:
test.cpp:5:9: warning: unused variable ‘i’ [-Wunused-variable]
    5 |     int i = 10;
```

↑  
変数 `int i` が使われていないという警告が表示されていることから、  
`LOG_TRACE("i = %d", i);` の行がコンパイラによって削除されているのが分かる  
「デバッグ時にはprintfのコメントアウトを外して...」とかやるより、ロガーを使った方が良い！

## 参考にしたコード

- <https://github.com/fmtlib/fmt>
- <https://github.com/gabime/spdlog/blob/v1.x/include/spdlog/spdlog.h>

## 追記

[組込みソフトウェア開発向けコーディング作法ガイド ESCR [C++言語版] Ver. 2.0](https://www.ipa.go.jp/sec/reports/20161018.html) にデバッグ用記述の例が2つ記載されていました

>問題発生時の原因を調査しやすい書き方にする。  
>デバッグオプション設定時のコーディング方法と、  
>リリースモジュールにログを残すためのコーディング方法を規定する。

```cpp:（a）デバッグ処理記述の切り分けに、マクロ定義を使用する方法
// DEBUG マクロは、コンパイル時に指定する
#ifdef DEBUG
fprintf(stderr, "var1 = %dn", var1);
#endif
```

```cpp:（b）assert マクロを使用する方法
// コンパイル時に、NDEBUG マクロが定義された場合、assert マクロは何もしない。
// 一方、 NDEBUG マクロが定義されない場合は、assert マクロに渡した式が偽の場合、ソースのファイル名と行位置を標準エラーに吐き出した後、異常終了する。
// マクロ名がDEBUG ではなく、NDEBUG であることに注意する。
// assert マクロは、処理系が<assert.h> で用意するマクロである。
// 以下を参考に、プロジェクトで異常終了のさせ方を検討し、処理系が用意しているマクロを使用するか、
// 独自のassert 関数を用意するかを決定する。
#ifdef NDEBUG
#define assert(exp) ((void)0)
#else
#define assert(exp) (void) ((exp)) || (_assert(#exp, __FILE__, __LINE__)))
#endif
void _assert(char *mes, char *fname, unsigned int lno)
{
    fprintf(stderr, "Assert : %s : %s(%d)\n", mes, fname, lno);
    fflush(stderr);
    abort();
}
```

## 終わりに

C++ なら本来は std::cout で出力した方が良いと思うんですが、  
書式を指定する際のコードがC言語よりも長くなってしまうのが何だかなあ...となりました。  
そこで std::format を使ってみると良さそう！ と試してみるも、  
[C++20でstd::formatを使おうとしたらコンパイラ側でまだ実装されていなかった](https://qiita.com/Yossy_Hal/items/e204c3fb8f86722ee9db)  
という事情で断念することに...  
std::cout への不満は <https://github.com/fmtlib/fmt#motivation> にも書かれていたりして、  
やっぱりみんな苦労してる点なんだなあと思いました
