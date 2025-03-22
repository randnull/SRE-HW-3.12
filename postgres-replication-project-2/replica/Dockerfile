FROM postgres:13-alpine

# Копируем конфигурацию восстановления
COPY postgresql.conf /usr/local/share/postgresql/postgresql.conf.sample

ENV PGPASSWORD=replicator_password

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh