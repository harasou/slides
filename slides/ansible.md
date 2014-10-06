class: center,middle
# 10 Minutes -Ansible-
![hakush](image/hakusi.png) 

---
class: center,middle
# @harasou

---
# Ansible とは

Python で書かれたサーバ構成管理ツール。

## 何ができる？

- SSH でログインして「任意コマンドの実行」
- SSH でログインして「サーバ設定」
    - パッケージ、コンフィグ、ユーザ、etc
    - 冪等性が確保された各種モジュールが提供されている。

## 何が必要？

- 対象ホスト
    - 通常モジュールによる設定は、Python スクリプト経由で実行されるので、 
      Python 2.4 or later (+ python-simplejson) が必要。
    - raw モジュール使用時は、Python もいらない。
- 管理ホスト
    - Python 2.6

---
# Glossary

- インベントリファイル
   - ホストの一覧が記載されたファイル（このファイルに記載がないホストに対しては操作はできない）
   - ディレクトリや実行ファイルでもよい。
- モジュール
   - 特定の機能（パッケージインストールやユーザ作成など）を提供するスクリプト
   - Ansible で実行できる全ての操作はモジュールにより提供されている。
- Playbook
   - 操作対象や操作内容をまとめて記述したファイル
   - YAML形式
- ansible.cfg
   - Ansible 全体に影響する設定を記載する。
   - hostfile、remote_user、private_key_file、ssh_args

---
# Commands

1. _ansible_：コマンドラインの引数で操作内容を指定
    ```
    ansible <host-pattern> -i INVENTORY -m MODULE_NAME -a MODULE_ARGS
    ```
1. _ansible-playbook_：操作内容を記述した playbook ファイルを指定
    ```
    ansible-playbook -i INVENTORY playbook.yml
    ```
1. _ansible-doc_：モジュールの説明を表示。便利 :-）
    ```
    ansible-doc MODULE_NAME
    ```
1. その他
    - _ansible-vault_  ：パスワードなどを記載した YAMLファイルを暗号化する
    - _ansible-galaxy_ ：Ansible Galaxy で公開されている Role を取得する
    - _ansible-pull_   ：リモートリポジトリの Playbook を自分に対し適用する

---
# ansible コマンド
- インベントリファイルに定義されたグループ「www」の対象ホスト一覧を表示
    ```
    ansible www -i hosts --list-hosts
    ```
- グループ「www」のホスト上の httpd を graceful
    ```
    ansible www -i hosts -a "/etc/init.d/httpd graceful"
    ```
- インベントリファイルで定義された全てのホスト上で「check.sh」を実行
    ```
    ansible all -i hosts -m script -a check.sh
    ```
- localhost に harasou でパスワードログインし、ホスト情報を取得
    ```
    ansible localhost -i hosts -ku harasou -m setup
    ```
- localhost に harasou でパスワードログインし、sudo 後 id コマンドを実行
    ```
    ansible localhost -i hosts -ku harasou -sK -a id
    ```

---
# ansible-playbook コマンド

- playbook を実行
    (インベントリに定義されていないホストには実行されない)
    ```
    ansible-playbook -i hosts playbook.yml
    ```
-  playbook.yml に定義されたタスクの一覧を表示
    ```
    ansible-playbook -i hosts --list-tasks playbook.yml
    ```
-  タスクごとの対象ホストを表示
    ```
    ansible-playbook -i hosts --list-hosts playbook.yml
    ```
-  タスクを一つずつ確認しながら実行
    ```
    ansible-playbook -i hosts --step playbook.yml
    ```

---
# Inventory
.center[ Docs : http://docs.ansible.com/intro_inventory.html ]
```
[www:children]
  www_group_A
  www_group_B

[www_group_A]
  www1.harasou.github.io
  www2.harasou.github.io
[www_group_B]
  www[100:150].harasou.github.io

[db]
  db01.harasou.github.io ansible_ssh_host=192.168.0.1
  db02.harasou.github.io ansible_ssh_host=192.168.0.2

[db:vars]
  ansible_ssh_user=harasou
  ansible_ssh_private_key_file=~/.ssh/id_rsa_harasou
```
.left-column.g4[
```
├─ansible.cfg
└─inventory
    ├─service_A
    ├─service_B
    └─service_C
```
]
.right-column.g12[
```
# 全てのサービスの全てのホストの uptime を確認
ansible all -i inventory -a "uptime"

# 各サービスのロール「www」のホストで、
# 　ポート80を listern しているプロセスを確認
ansible www -i inventory -m shell -a "netstat -tlnp|grep :80"
```
]

---
# Module

.center[
Docs : http://docs.ansible.com/modules_by_category.html
]

| モジュール   | 役目                                                                            |冪等性      |
|:-------------|:--------------------------------------------------------------------------------|:-----------|
| shell        |シェル上でコマンドを実行                                                         |creates     |
| file         |ファイルやディレクトリの作成、パーミッション                                     |自動        |
| lineinfile   |指定ファイルの行単位の書き換え                                                   |半自動      |
| copy         |ファイルの丸コピー                                                               |自動        |
| template     |テンプレートを利用したファイルのコピー                                           |自動        |
| synchronize  |要するに rsync                                                                   |自動        |
| get_url      |指定URLからダウンロード                                                          |自動        |
| yum          |yum コマンド。enablerepo やdisablerepo はもちろん rpm から直接インストールも可能 |自動        |
| service      |service コマンド。chkconfig も兼ねる                                             |自動        |

---
# Playbook

---
# ansible.cfg
.center[
Docs : http://docs.ansible.com/intro_configuration.html
]
