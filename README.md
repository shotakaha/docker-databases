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

## バージョン確認

```console
$ docker compose exec db mariadb --version
mariadb  Ver 15.1 Distrib 10.11.11-MariaDB, for debian-linux-gnu (aarch64) using  EditLine wrapper
```

## シェル起動

```console
$ docker compose exec db bash
root@8e49936894e2:/# 
```

## 終了

```console
$ docker compose down
[+] Running 2/2
 ✔ Container docker-mariadb-db-1   Removed
 ✔ Network docker-mariadb_default  Removed
 ```