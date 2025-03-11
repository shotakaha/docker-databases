# docker-mariadb

docker compose mariadb

## イメージ

- https://hub.docker.com/_/mariadb

## 環境変数

| MySQL | MariaDB | 説明 | デフォルト値 |
|---|---|---|---|
| `MYSQL_ROOT_PASSWORD` | `MARIADB_ROOT_PASSWORD` | ルートユーザーのパスワード | なし（必須） |
| `MYSQL_ALLOW_EMPTY_PASSWORD` | `MARIADB_ALLOW_EMPTY_ROOT_PASSWORD` | ルートユーザーのパスワードなしを許可 | なし |
| `MYSQL_RANDOM_ROOT_PASSWORD` | `MARIADB_RANDOM_ROOT_PASSWORD` | ルートユーザーのパスワードをランダム生成 | なし |
| `MYSQL_ROOT_HOST` | `MARIADB_ROOT_HOST` | ルートユーザーの接続を許可するホスト名 | `localhost` |
| `MYSQL_TCP_PORT` | `MARIADB_TCP_PORT` | データベースに接続するポート番号 | `3306` |
| `MYSQL_DATABASE` | `MARIADB_DATABASE` | 自動生成するデータベース名 | なし |
| `MYSQL_INITDB_SKIP_TZINFO` | `MARIADB_INITDB_SKIP_TZINFO` | TZテーブルの初期化をスキップ | なし |
| `MYSQL_USER` | `MARIADB_USER` | 一般ユーザーを作成 | なし |
| `MYSQL_PASSWORD` | `MARIADB_PASSWORD` | 一般ユーザーのパスワード | なし |
| `MYSQL_LOG_CONSOLE` | `MARIADB_LOG_CONSOLE` | エラーログを標準出力に設定 | なし |
| | `MARIADB_AUTO_UPGRADE` | データベースの自動アップグレードを実行 | `no` |
| | `MARIADB_DISABLE_UPGRADE_BACKUP` | アップグレード時のバックアップ作成を無効化 | `no` |
| | `MARIADB_CREATE_DATABASE_IF_NOT_EXISTS` | `MARIADB_DATABASE`の設定がない場合でもデータベースを作成 | `no` |

- MySQLとMariaDBは基本的に同じ環境変数を設定できます
  - MariaDBには追加機能があります
  - MariaDB独自の環境変数はMySQLでは利用できません
- 環境変数のプレフィクスはそれぞれ`MYSQL_`と`MARIADB_`です
  - MySQLの環境変数は基本的にMariaDBでも動作します

## 起動

```console
$ docker compose up -d
docker compose up -d
[+] Running 2/2
 ✔ Network docker-mariadb_default  Created
 ✔ Container docker-mariadb-db-1   Started
```

`docker compose up`でコンテナを起動します。
このとき`-d`オプション（デタチモード）をつけます。

## 作業パスの確認

```console
$ docker compose exec db pwd
/workspace
```

`pwd`を実行し、作業パスを確認します。
作業パスは、`services.working_dir`で変更できます。
デフォルトは`/`（ルートディレクトリ）です。

## シェル起動

```console
$ docker compose exec db bash
root@8e49936894e2:/workspace# 
```

## バージョン確認

```console
root@e6c5df337cd7:/workspace# mariadb --version
mariadb  Ver 15.1 Distrib 10.11.11-MariaDB, for debian-linux-gnu (aarch64) using  EditLine wrapper
```

コンテナのシェルの中で`mariadb --version`を実行し、バージョンを確認します。

```console
root@e6c5df337cd7:/workspace# mysql --version
mysql  Ver 15.1 Distrib 10.11.11-MariaDB, for debian-linux-gnu (aarch64) using  EditLine wrapper
```

`mysql`コマンドもありました。中身はMariaDBでした。

## 終了

```console
$ docker compose down
[+] Running 2/2
 ✔ Container docker-mariadb-db-1   Removed
 ✔ Network docker-mariadb_default  Removed
```

```console
$ docker compose down -v
[+] Running 3/3
 ✔ Container docker-mariadb-db-1   Removed
 ✔ Volume docker-mariadb_db_data   Removed
 ✔ Network docker-mariadb_default  Removed
```

`-v`オプションでボリュームも削除できます。

## コンテナの確認

```console
$ docker container ls
CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS              PORTS                  NAMES
9ec9772778e2   mariadb:10.11          "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp               docker-mariadb-db-1
```

```console
$ docker volume ls
DRIVER    VOLUME NAME
local     docker-mariadb_db_data
```

## データベースの操作

```console
$ docker compose exec db bash
root@9ec9772778e2:/workspace# mariadb --user=testuser --database=testdb -p
Enter password:  // testpassword
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.11.11-MariaDB-ubu2204 mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [testdb]> 
```

コンテナのシェルを起動し、`mariadb`コマンドを実行する。
`--user=ユーザー名`に`MARIADB_USER`、
`--database=データベース`に`MARIADB_DATABASE`の値を指定します。
パスワードは`MARIADB_PASSWORD`を入力します。

## データベースのバックアップ

```console
$ docker compose exec db bash
root@e8f060e53c00:/workspace# mariadb-dump --print-defaults
mariadb-dump would have been started with the following arguments:
--socket=/run/mysqld/mysqld.sock 

root@e8f060e53c00:/workspace# mysql --user testuser -p testdb > backup.sql
Enter password: // testpass
root@e8f060e53c00:/workspace# exit

$ docker compose cp db:/workspace/backup.sql .
```

`mariadb-dump`もしくは`mysqldump`を使って、データベースをダンプします。
上記のサンプルでは、作業パスにバックアップを作成し、
`docker compose cp`でホストに転送しています。