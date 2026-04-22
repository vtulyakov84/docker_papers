## Работа с образом PostgreSQL `registry.altlinux.org/c10f/postgresql`

Обзор конфигурации контейнера:

* Запуск контейнера `registry.altlinux.org/c10f/postgresql`

* Определение каталога для хранения данных PostgreSQL

* Определение файлов конфигурации `postgresql.conf`, `pg_hba.conf`

### 1. Файловая структура проекта
```bash
[user@vbox postgres]$ tree .
.
├── config
│   ├── pg_hba.conf
│   └── postgresql.conf
├── Dockerfile
└── scripts
    └── initdb.sql
```

`./config/pg_hba.conf`     - конфигурация аутентификации и доступа к базе данных
`./config/postgresql.conf` - конфигурация основных настроек сервера
`./Dockerfile`             - сценарий создание контейнера
`./scripts/`               - каталог скриптов инициализации базы данных  

### 2. Запуск контейнера

#### 2.1. Сбор общей информации
```bash
# При отсутствии образа в локальном хранилище хоста
# он будет скачан с указанного репозитория
[user@... postgres]$ docker run -it registry.altlinux.org/c10f/postgresql bash
bash-4.4$ 
```

```bash
# Определяем:
# - текущего пользователя
bash-4.4$ whoami
postgres

# - используемую версию PostgreSQL
bash-4.4$ postgres --version
postgres (PostgreSQL) 16.11

# - домашний каталог
bash-4.4$ pwd
/var/lib/pgsql

# - содержимое домашнего каталога
bash-4.4$ ls -la
total 24
drwx------ 4 postgres postgres 4096 Dec  2 10:52 .
drwxr-xr-x 1 root     root     4096 Dec  2 10:52 ..
-rw-r--r-- 1 postgres postgres   70 Dec  2 10:52 .bash_profile
drwx------ 2 postgres postgres 4096 Nov 13 05:21 backups
drwx------ 2 postgres postgres 4096 Nov 13 05:21 data

# - содержание файла `.bash_profile`
bash-4.4$ cat .bash_profile 
PGLIB=/usr/share/pgsql
PGDATA=/var/lib/pgsql/data
export PGLIB PGDATA

# - переменные программного окружения
bash-4.4$ printenv
HOSTNAME=7d151d3da7c0
PWD=/var/lib/pgsql
HOME=/var/lib/pgsql
TERM=xterm
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/printenv

# - открыт ли порт прослушки
bash-4.4$ ss -tulpn | grep 5432
Netid    State    Recv-Q    Send-Q         Local Address:Port         Peer Address:Port    Process 
```

В домашнем каталоге пользователя `postgres` (`/var/lib/pgsql`)будем хранить файлы конфигурации работы PostgreSQL `postgresql.conf`, `pg_hba.conf`.

#### 2.2. Сбор информации о работе postgres

```bash
# Отобразить справку о вариантах работы с командной `postgres`
bash-4.4$ postgres --help
```

> **NoteBene!** При подключении к базе данных из контейнера выходит ошибка 
bash-4.4$ psql -d postgres -p 5432
psql: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: No such file or directory
        Is the server running locally and accepting connections on that socket?
Позже проверить подключение с командной строки внутри контейнера, после сборки образа с файлами конфигурации.

### 3. Определение параметров для сборки своего образа

#### 3.1. Файл конфигурации работы сервера `./postgresql.conf`:
```ini
listen_addresses = '*'
port = 5432
max_connections = 100
shared_buffers = 128MB
dynamic_shared_memory_type = posix
max_wal_size = 1GB
min_wal_size = 80MB
log_timezone = 'Etc/UTC'
datestyle = 'iso, mdy'
timezone = 'Etc/UTC'
lc_messages = 'en_US.utf8'
lc_monetary = 'en_US.utf8'
lc_numeric = 'en_US.utf8'
lc_time = 'en_US.utf8'
default_text_search_config = 'pg_catalog.english'
```

#### 3.2. Файл конфигурации аутентификации сервера `pg_hba.conf`:
```
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
host    all             all             0.0.0.0/0               md5
```

#### 3.3. Место хранения файлов данных PostgreSQL
В связи с тем, что после останова контейнера он не сохраняет состояния и любые изменения при повторном запуске пропадают, хранить файлы базы данных необходимо на хостовой системе, с которой выполняется запуск контейнера.

```bash
# 1. Убедитесь, что каталог, в котором будут храниться
#    файлы базы данных отсутствует
[user@... docker_papers]$ sudo ls -la /var/lib/postgresql/data
ls: невозможно получить доступ к '/var/lib/postgresql/data': Нет такого файла или каталога

# 2. Создаем каталог для хранения файлов базы данных
[user@... docker_papers]$ sudo mkdir -p /var/lib/postgresql/data
```

### 4. Сборка своего образа на базе `registry.altlinux.org/c10f/postgresql`
```Dockerfile
FROM registry.altlinux.org/c10f/postgresql

# Определяем переменные окружения
# ENV

# Копируем файлы конфигурации PostgreSQL
# в домашний каталог контейнера

```


> проверить останавливается ли контейнер после запуска, без обращения к bash, если останавливается значит postgres не запущен как служба.

> проверить состояние работы службы postgres.