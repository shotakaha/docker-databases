# docker-mariadb

docker compose mariadb

## イメージ

- https://hub.docker.com/_/mariadb

## 環境変数

- `MARIADB_ROOT_PASSWORD` : ルートユーザーのパスワード
- `MARIADB_DATABASE` : データベース名
- `MARIADB_USER` : データベースのユーザー名
- `MARIADB_PASSWORD` : データベースのパスワード

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

## バージョン確認

```console
$ docker compose exec db mariadb --version
mariadb  Ver 15.1 Distrib 10.11.11-MariaDB, for debian-linux-gnu (aarch64) using  EditLine wrapper
```

`docker compose exec`で`mariadb --version`を実行し、バージョンを確認します。
コンテナ名は`services`で設定した値（`db`）を指定しています。

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