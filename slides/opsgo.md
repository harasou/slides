class: center,middle
ノンプログラマによる
# Ops から見た golang

---
class: center,middle
# @harasou

---
background-image: url(image/gophermega.jpg)
class: right,bottom
ゴーファー(ホリネズミ)

---
# golang とは

2009年に Google により発表されたオープンソースのプログラミング言語。

golang は、Linux、Mac、Windows、[NaCl](http://ja.wikipedia.org/wiki/Google_Native_Client)
などで動作する開発言語で、Android携帯上でも動作する。2014年6月に 1.3 をリリース。

名だたる開発陣。

## Ken hompson  .small[(ケン・トンプソン)]
- C言語の開発者

## Rob Pike .small[(ロブ・パイク)]
- UTF-8 の開発者

## Brad Fitzpatrick .small[(ブラッド・フィッツパトリック)]
- memcached の開発者

---
# golang の特徴

## クロスコンパイルのサポート

コンパイル時に OS と アーキテクチャを指定することで、その環境に合わせたバイナリを生成できる
([サポート対象](http://golang.org/doc/install/source#environment))。
```sh
GOOS=linux GOARCH=amd64 go build hello.go
```

## 1つのファイルで動く

`go build` で生成されるバイナリは、ライブラリがスタティックリンクされたネイティブコード。

Ruby や PHP のようなスクリプト言語ではないのでインタプリンタも不要。
必要なライブラリもリンクされているため依存も気にする必要がない。

## シンプルな言語

ノンプログラマなため文法等の特徴はスルー :-P

---
# golang の事例

## Vitess (YouTube) [ref](https://github.com/youtube/vitess/)
MySQL のロードバランサ。YouTube の全MySQLクエリをさばいている。

## Docker [ref](https://www.docker.com/)
言わずと知れた、コンテナ型の仮想環境構築ツール。

## Packer [ref](http://www.packer.io/)
仮想環境へのOSインストールやテンプレート作成を行うツール。

## mackerel (Hatena) [ref](https://mackerel.io/)
サーバ管理ツール。エージェントに golang が使われている。

## pt (monochromegane) [ref](http://blog.monochromegane.com/blog/2014/01/16/the-platinum-searcher/)
ポスト`ag`な高速検索ツール。

---
# Install

## 新規インストール
MacOSX の場合は、brew で。

```sh
brew update
brew install go --cross-compile-common
```
.small[
- cross-compile-common：linux, windows, darwin 用のクロスコンパイル環境を準備する。
- cross-compile-all   ：全てのクロスコンパイル環境を準備する。[refs](https://github.com/Homebrew/homebrew/)
]

## インストール済みの場合
オプションをつけずにインストールしていた場合は、下記コマンドでクロスコンパイルの環境が整う
([gox](https://github.com/mitchellh/gox)を使用すると便利)。

```sh
brew update
brew upgrade go
```
```sh
cd "$(go env GOROOT)/src"
GOOS=linux GOARCH=386   ./make.bash  # linux32bit 環境用
GOOS=linux GOARCH=amd64 ./make.bash  # linux64bit 環境用
```

---
# Cross Compile -1-

.left.g8[
## hostname.go
```go
package main

import (
  "fmt"
  "os"
)

func main() {
  hostname, _ := os.Hostname()
  fmt.Println(hostname)
}
```
]

## go build
```
GOOS=linux GOARCH=386   go build -o hostname_linux_386   hostname.go
GOOS=linux GOARCH=amd64 go build -o hostname_linux_amd64 hostname.go
```
```
hostname_linux_386:   ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), statically linked, not stripped
hostname_linux_amd64: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```

---
# Cross Compile -2-

## Ansible Playbook

```yaml
---
- hosts: www001
  gather_facts: no
  remote_user: harasou
  name: 32bit マシンで hostname_linux_386 を実行する
  tasks:
    - copy: src=hostname_linux_386 dest=~/hostname_linux_386 mode=0755
    - raw: ./hostname_linux_386

- hosts: www201
  gather_facts: no
  remote_user: harasou
  name: 64bit マシンで hostname_linux_amd64 を実行する
  tasks:
    - copy: src=hostname_linux_amd64 dest=~/hostname_linux_amd64 mode=0755
    - raw: ./hostname_linux_amd64
```

---
# Cross Compile -3-
## ansible-playbook
```
$ ansible-playbook -i inventory/ playbook.yml

PLAY [www001] *****************************************************************

TASK: [copy src=hostname_linux_386 dest=~/hostname_linux_386 mode=0755] *******
changed: [www001]

TASK: [raw ./hostname_linux_386] **********************************************
*ok: [www001]

PLAY [www201] *****************************************************************

TASK: [copy src=hostname_linux_amd64 dest=~/hostname_linux_amd64 mode=0755] ***
changed: [www201]

TASK: [raw ./hostname_linux_amd64] ********************************************
*ok: [wwww02]

PLAY RECAP ********************************************************************
wwww001    : ok=2    changed=1    unreachable=0    failed=0
wwww202    : ok=2    changed=1    unreachable=0    failed=0
```
