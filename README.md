# docker-mariadb

docker compose mariadb/mysql + adminer

## Images

- [MariaDB + Adminer](./mariadb_adminer/README.md)
- [MySQL + Adminer](./mysql_adminer/README.md)
- [PostgreSQL + Adminer](./postgres_adminer/README.md)

Database Management Tools

- Adminer: https://hub.docker.com/_/adminer
- phpMyAdmin: https://hub.docker.com/_/phpmyadmin

## Usage

```console
$ docker compose up -d
$ open http://localhost:8080
$ docker compose down -v
```

- `MariaDB`（もしくは`MySQL`）コンテナと`Adminer`コンテナを起動します
- `http://localhost:8080`で`Adminer`アプリにアクセスし、データベースを操作します
- コンテナ終了時にはボリュームを削除します



## Environments

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

- MySQLとMariaDBは基本的に同じ環境変数を利用できます
  - MariaDBには追加機能があります
  - MariaDB独自の環境変数はMySQLでは利用できません
- 環境変数のプレフィクスはそれぞれ`MYSQL_`と`MARIADB_`です
  - MySQLの環境変数は基本的にMariaDBでも動作します

## コンテナを起動する

```console
$ docker compose up -d
[+] Running 4/4
 ✔ Network docker-mariadb_default      Created
 ✔ Volume "docker-mariadb_db_data"     Created
 ✔ Container docker-mariadb-adminer-1  Started
 ✔ Container docker-mariadb-db-1       Started
```

`docker compose up -d`で`MariaDB`コンテナと`Adminer`コンテナを起動します。
起動時には`-d`オプション（デタッチモード）を使用します。

## ログを確認する

```console
$ docker compose logs
...
db-1  | 2025-03-18 14:10:39 0 [Note] mariadbd: ready for connections.
db-1  | Version: '10.11.11-MariaDB-ubu2204'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution

$ docker compose logs adminer
adminer-1  | [Tue Mar 18 22:41:10 2025] PHP 8.4.5 Development Server (http://[::]:8080) started
adminer-1  | [Tue Mar 18 22:42:52 2025] [::ffff:192.168.65.1]:49820 Accepted
adminer-1  | [Tue Mar 18 22:42:52 2025] [::ffff:192.168.65.1]:49820 [200]: GET /
adminer-1  | [Tue Mar 18 22:42:52 2025] [::ffff:192.168.65.1]:49820 Closing
```

デタッチモードで起動した場合でも
`docker compose logs コンテナ名`で標準出力のログを確認できます。
コンテナ名を指定するとコンテナごとのログを取得できます。

## データベースを初期化する

```yaml
services:
  db:
    image: madiadb:10.11
    volumes:
      - ./backups/:/docker-entrypoint-initdb.d/
```

エントリーポイントを利用してデータベースを初期化します。
MariaDBやMySQLなどのデータベース系の公式イメージでは、
コンテナの起動時に`/docker-entrypoint-initdb.d/`の中にある
ファイルを読み込むように設定されています。

対象となるファイルは`.sql`や`.sql.gz`、`.sh`です。
複数のファイルがある場合はABC順に読み込まれます。
この仕様を利用して、ファイル名に数字の接頭辞をつけることで読み込む順番を制御できます。

この`compose.yaml`の設定では
ホストPCの`./backups/`の中にデータベースを保存することを前提に、
`/docker-entrypoint-initdb.d`にマウントしています。

注意点として、この処理はコンテナ内にデータベースがないときだけ実行されます。
そのためボリュームを永続化している場合は、事前に削除しておく必要があります。

## データベースを確認する

`http://localhost:8080`で`Adminer`コンテナに接続します。
もしくは、下記のように`db`コンテナに接続し、直接操作することもできます。

## データベースに接続する

```console
// コンテナを起動する
$ docker compose up -d

// コンテナに接続しbashを起動する
$ docker compose exec db bash

// データベースに接続する
[root@random:/workspace]# mariadb -u testuser -D testdb -p
Enter password:    # testpassと入力
Server version: 10.11.11-MariaDB-ubu2204 mariadb.org binary distribution

MariaDB [testdb]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| testdb             |
+--------------------+
2 rows in set (0.001 sec)

MariaDB [testdb]> use testdb;
Database changed

MariaDB [testdb]> show tables;
+-----------------------------------+
| Tables_in_testdb                  |
+-----------------------------------+
| カラム名                           |
| .......                           |
+-----------------------------------+
139 rows in set (0.001 sec)

MariaDB [testdb]> \q
Bye

// コンテナからログアウトする
[root@random:/workspace]# exit

// コンテナを終了する
$ docker compose down -v
```

`docker compose exec db bash`でコンテナのbashを起動します。
あとはコンテナ内でデータベースを操作をします。

データベースに接続する時は、環境変数の
`MARIADB_USER`、
`MARIADB_DATABASE`、
`MARIADB_PASSWORD`
に設定した値を入力します。

コンテナを終了するときは`-v`オプションで
ボリュームを削除します。

---

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


