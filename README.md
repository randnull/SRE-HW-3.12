# Задание

# Проблемы

Запустим docker-compose и увидим, что replica постоянно ждет ввода пароля, а в конце падает:


Для начала найдем быстрое решение - просто поменяем параметр с md5 на trust в pg_hba.conf (это конечно небезопасно, так как мы отключили аутентификацию)

Зато это работает:)
```
RUN echo "host replication replicator 0.0.0.0/0 trust" >> /usr/local/share/postgresql/pg_hba.conf.sample
```

Проверим скорость репликаций:

| pid | usesysid | usename    | application_name | client_addr | client_hostname | client_port | backend_start               | backend_xmin | state     | sent_lsn   | write_lsn  | flush_lsn  | replay_lsn | write_lag      | flush_lag      | replay_lag     | sync_priority | sync_state | reply_time                   |
|-----|----------|-----------|------------------|-------------|-----------------|-------------|-----------------------------|--------------|-----------|------------|------------|------------|------------|----------------|----------------|----------------|---------------|------------|------------------------------|
| 563 | 16395    | replicator | walreceiver      | 172.18.0.3  |                 | 51522       | 2025-03-22 15:01:20.680994+00 |              | streaming | 0/53DD50A0 | 0/53DD50A0 | 0/53DD50A0 | 0/53DD50A0 | 00:00:00.0013  | 00:00:00.00235 | 00:00:00.00282  | 0             | async      | 2025-03-22 15:03:53.53187+00  |
| 562 | 16395    | replicator | walreceiver      | 172.18.0.4  |                 | 49700       | 2025-03-22 15:01:20.606135+00 |              | streaming | 0/53DD50A0 | 0/53DD50A0 | 0/53DD50A0 | 0/53DD50A0 | 00:00:00.001239 | 00:00:00.00226 | 00:00:00.002733 | 0             | async      | 2025-03-22 15:03:53.531785+00 |

Как видно, задержек составляет 2-3 милисекунды.

Второй способ починить - вернуть аутентификацию (md5), и передать пароль вручную 

Например передадим пароль через docker env:
ENV PGPASSWORD=replicator_password
либо использовать ~/.pgpass, как советует дока (https://postgrespro.ru/docs/postgresql/9.6/libpq-envars)

# Поиск ошибок

В Dockerfile для master вручную записываеются данные, хотя есть файл postgres.conf
```
Настройка PostgreSQL для репликации
RUN echo "wal_level = replica" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "max_wal_senders = 3" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "wal_keep_size = 512MB" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "listen_addresses = '*'" >> /usr/local/share/postgresql/postgresql.conf.sample
```

исправим это на 
```
COPY postgresql.conf /usr/local/share/postgresql/postgresql.conf.sample
```

Также этим мы решили проблему с подлючением реплик, посколько параметр max_wal_senders, отвечающий за максимальное количество одновременных подключений от резервных серверов или клиентов потокового базового резервного копирования (https://www.postgresql.org/docs/current/runtime-config-replication.html), рекомендуют устанавливать больше количества реплик. (изначально 3, теперь 4)

Строка 
```
# RUN echo "host all all" >> /usr/local/share/postgresql/pg_hba.conf.sample
```

Не очень понятна, так как вызывает ошибку при запуске, из-за отсуствия метода аутентификации и диапозона ip.

Также заменим wal_keep_segments на wal_keep_size = 512MB