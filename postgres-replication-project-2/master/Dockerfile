FROM postgres:13-alpine

# Копируем скрипт инициализации
COPY init.sql /docker-entrypoint-initdb.d/

# Настройка PostgreSQL для репликации
COPY postgresql.conf /usr/local/share/postgresql/postgresql.conf.sample

# Разрешаем подключение реплики
RUN echo "host replication replicator 0.0.0.0/0 md5" >> /usr/local/share/postgresql/pg_hba.conf.sample
