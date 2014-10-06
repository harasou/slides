class: center, middle
# Apache module ことはじめ
Apache のモジュールを理解するために必要な基礎的なこと

---
class: center, middle

# @harasou

---
#  agenda
1. Apache の動作
   - マスタプロセス
   - 子プロセス
1. module の動作
   - hook
   - module のサンプル
   - 代表的な hook 登録関数
   - hook 登録関数のプロトタイプ
   - hook 関数の返り値
   - hook 関数の引数
1. 具体例
1. 参考サイト

---
class: center, middle

# Apache の動作

まずは簡単に、Apache 本体の動作について

---
## マスタプロセス

httpd を起動すると、親プロセス（マスタプロセス）により、以下のような処理が行われる。

![fig1.png](image/fig1.png)

--
1. 設定の初期化
    - コンフィグファイルの読み込み
    - ログファイルのオープン
--
1. 子プロセスの起動

--
1. 子プロセスの終了を待つ

--
1. 子プロセス数の調整

---
## 子プロセス（スレッド）

生成された子プロセスの処理のながれ。

--
1. プロセスの初期化 (conn_rec の生成)

--
1. リクエストの受付 (request_rec の生成)

--
1. コンテンツマッピング

--
1. アクセス制御(AAA)

--
1. コンテンツの配信

--
1. ログの記録


---
class: center, middle
# module の動作
module が動作するタイミング
---
## hook
Apache では、各処理に hook を仕込むことができる。

> フック（Hook）は、プログラム中の特定の箇所に、利用者が独自の処理を追加できるようにする仕組みである。(Wikipedia)

module の動作は、各処理に存在する hook に、自分が行いたい処理を記述した hook 関数を登録することで実現される。

.left-column.small[
1. 設定の初期化
    - コンフィグファイルの読み込み
    - ログファイルのオープン
1. 子プロセスの起動
    1. プロセスの初期化 (conn_rec の生成)
    1. リクエストの受付 (request_rec の生成)
    1. コンテンツマッピング
    1. アクセス制御(AAA)
    1. コンテンツの配信
    1. ログの記録
1. 子プロセスの終了を待つ
1. 子プロセス数の調整
]
.right-column.g8[
![fig2.png](image/fig2.png)
]
---
## module のサンプル - apxs コマンド

```console
[harasou@build64-users module]$ apxs -g -n hello
Creating [DIR]  hello
Creating [FILE] hello/Makefile
Creating [FILE] hello/modules.mk
Creating [FILE] hello/mod_hello.c
Creating [FILE] hello/.deps
```

---
## module のサンプル - mod_hello.c
```c
#include "httpd.h"
#include "http_config.h"
#include "http_protocol.h"
#include "ap_config.h"

static int hello_handler(request_rec *r)
{
    if (strcmp(r->handler, "hello")) {
        return DECLINED;
    }
    r->content_type = "text/html";

    if (!r->header_only)
        ap_rputs("The sample page from mod_hello.c\n", r);
    return OK;
}

static void hello_register_hooks(apr_pool_t *p)
{
    ap_hook_handler(hello_handler, NULL, NULL, APR_HOOK_MIDDLE);
}

module AP_MODULE_DECLARE_DATA hello_module = {
    STANDARD20_MODULE_STUFF,
    NULL,                  /* create per-dir    config structures */
    NULL,                  /* merge  per-dir    config structures */
    NULL,                  /* create per-server config structures */
    NULL,                  /* merge  per-server config structures */
    NULL,                  /* table of config file commands       */
    hello_register_hooks  /* register hooks                      */
};
```

---
## 代表的な hook 登録関数

ref: [core.c](https://github.com/apache/httpd/blob/eb6f18bd5c6d25b0ff38a9add09dbd243e3a8609/server/core.c#L5161-L5221)

|hook 登録関数|実行フェーズ|フックの種類|
|:-----------|:---------|:---------|
|ap_hook_pre_config|設定の初期化|RUN_ALL|
|ap_hook_open_logs|設定の初期化|RUN_ALL|
|ap_hook_post_config|設定の初期化|RUN_ALL|
|ap_hook_child_init|プロセスの初期化|RUN_ALL|
|ap_hook_access_checker|アクセス制御|RUN_ALL|
|ap_hook_check_user_id|アクセス制御|RUN_ALL|
|ap_hook_auth_checker|アクセス制御|RUN_FIRST|
|ap_hook_handler|コンテンツの配信|RUN_FIRST|

.small[
- VOID
    - フックに登録されている hook関数がすべて呼び出される。モジュールの初期化などに利用。
- RUN_FIRST
    - フックを順番に呼び出していき，DECLINE 以外のステータス（＝成功 or エラー）を返した場合にそこで呼び出しチェーンがストップする。
- RUN_ALL
    - エラーが返されない限りすべての hook関数が呼び出される。コンフィグファイルの読み込みなどに利用。
]

---
## hook 登録関数のプロトタイプ

```c
static void hello_register_hooks(apr_pool_t *p)
{
    ap_hook_handler(hello_handler, NULL, NULL, APR_HOOK_MIDDLE);
}
```

`ap_hook_***(func, pre, suc, pos);`

- func：登録する hook 関数
- pre：登録する hook 関数より前に実行されているべきモジュールのリスト
- suc：登録する hook 関数より後に実行されているべきモジュールのリスト
- pos：登録する hook 関数を実行する大体の位置．
    - APR_HOOK_REALLY_FIRST
    - APR_HOOK_FIRST
    - APR_HOOK_MIDDLE
    - APR_HOOK_LAST
    - APR_HOOK_REALLY_LAST


---
## hook 関数の引数

```c
static int hello_handler(request_rec *r)
{
    if (strcmp(r->handler, "hello")) {
        return DECLINED;
    }
    r->content_type = "text/html";

    if (!r->header_only)
        ap_rputs("The sample page from mod_hello.c\n", r);
    return OK;
}
```

.left-column[
## [request_rec 構造体](https://github.com/apache/httpd/blob/20656c3b77cc548b59fea3bde5e2b7705d71c427/include/httpd.h#L783-L1047)

- HTTP リクエストを表現した構造体
- 主なメンバ
    - r->connection
    - r->handler
    - r->method
    - r->filename
    - r->args
    - r->useragent_ip
    - r->pool

]

.right-column[
## [conn_rec 構造体](https://github.com/apache/httpd/blob/20656c3b77cc548b59fea3bde5e2b7705d71c427/include/httpd.h#L1075-L1184)

- コネクションを表現した構造体
- 主なメンバ
    - c->client_ip
    - c->client_addr
]

---
## hook 関数の返り値

```c
static int hello_handler(request_rec *r)
{
    if (strcmp(r->handler, "hello")) {
        return DECLINED;
    }
    r->content_type = "text/html";

    if (!r->header_only)
        ap_rputs("The sample page from mod_hello.c\n", r);
    return OK;
}
```

- OK：処理成功
- DECLIEND：次のモジュールにおまかせ
- HTTP_*
    - HTTP_NOT_FOUND
    - HTTP_FORBIDDEN
    - HTTP_INTERNEL_SERVER_ERROR

---

# module example

- [mod_extract_forwarded](https://github.com/matsumoto-r/mod_extract_forwarded_for_2.4/blob/master/mod_extract_forwarded.c)
- [mod_geoip](https://github.com/Oneiroi/mod_geoip/blob/master/mod_geoip.c)
- [mod_mruby](https://github.com/matsumoto-r/mod_mruby/blob/master/src/mod_mruby.c)

---
# reference document

- https://github.com/apache/httpd/tree/2.4.x
- http://blog.matsumoto-r.jp/?p=1625
- http://d.hatena.ne.jp/dayflower/20081029/1225266220
- http://www.slideshare.net/shebang/apache-module-presentation
