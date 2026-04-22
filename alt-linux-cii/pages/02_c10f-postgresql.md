## Работа с образом PostgreSQL `registry.altlinux.org/c10f/postgresql`

Обзор конфигурации контейнера:

* Запуск контейнера `registry.altlinux.org/c10f/postgresql`

* Определение каталога для хранения данных PostgreSQL

* Определение файлов конфигурации `postgresql.conf`, `pg_hba.conf`

### 1. Файловая структура проекта
```bash
[user@vbox postgres]$ tree .
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

# Домашний каталог
bash-4.4$ pwd
/var/lib/pgsql

# Используемую версию PostgreSQL
bash-4.4$ postgres --version
postgres (PostgreSQL) 16.11
```
