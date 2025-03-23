# docker-mariadb

docker compose MariaDB

## Images

- MariaDB: https://hub.docker.com/_/mariadb

## Usage

### Start services

```console
$ docker compose up -d
[+] Running 3/3
 ✔ Network mariadb_default   Created
 ✔ Volume "mariadb_db_data"  Created
 ✔ Container mariadb-db-1    Started
```

### Check services

```console
$ docker compose ps
NAME           IMAGE           COMMAND                  SERVICE   CREATED          STATUS          PORTS
mariadb-db-1   mariadb:10.11   "docker-entrypoint.s…"   db        58 seconds ago   Up 58 seconds   3306/tcp
```

### Use MariaDB

```console
$ docker compose exec db bash

[root@container:/workspace]# mariadb --version
mariadb  Ver 15.1 Distrib 10.11.11-MariaDB, for debian-linux-gnu (aarch64) using  EditLine wrapper

[root@container:/workspace]# mariadb -u testuser -D testdb -p
Enter password:     # testpass
MariaDB [testdb]> show databases;

MariaDB [testdb]> \q
Bye

[root@container:/workspace]# exit

$
```

### Quit services

```console
$ docker compose down -v
[+] Running 3/3
 ✔ Container mariadb-db-1   Removed
 ✔ Volume mariadb_db_data   Removed
 ✔ Network mariadb_default  Removed
```
