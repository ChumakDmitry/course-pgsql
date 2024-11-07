### Домашнее задание №10

1. Установить 16 ПГ.
2. Залить средние Тайские перевозки

```wget https://storage.googleapis.com/thaibus/thai_medium.tar.gz && tar -xf thai_medium.tar.gz && psql < thai.sql```

Сделаем выборку всех билетов

```
thai=# select count(*) from book.tickets;
  count   
----------
 53997475
(1 row)
```

3. Рядом поднять кластер 17 версии

Установил postgres 17 версии, кластер поднялся автоматически. Сделал настройки как в лекции

```
psql -c "ALTER SYSTEM SET wal_level = logical;"
pg_ctlcluster 16 main stop && pg_ctlcluster 16 main start

-- 2 cluster
psql -p 5434 -c "ALTER SYSTEM SET wal_level = logical;"
pg_ctlcluster 17 main stop && pg_ctlcluster 17 main start
```

4. Протестировать скорость онлайн вариантов миграции (логическая репликация, postgres_fdw, pg_dump/pg_restore)

а) Логическая репликация

```
psql -d thai
CREATE PUBLICATION test_pub FOR TABLES IN SCHEMA book;

-- копируем схему
pg_dumpall -s > schema.sql
psql -p 5434 < schema.sql
psql -p 5434 -d thai

-- для удобного подсчёта времени выведем текущее время до создания подписки
SELECT now();
CREATE SUBSCRIPTION test_sub 
CONNECTION 'host=localhost port=5432 user=postgres password=secret$123 dbname=thai' 
PUBLICATION test_pub WITH (copy_data = true);
             now              
------------------------------
 2024-11-07 09:33:23.45671+03
(1 row)

thai=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 24708
subname               | test_sub
worker_type           | table synchronization
pid                   | 5104
leader_pid            | 
relid                 | 24631
received_lsn          | 
last_msg_send_time    | 2024-11-07 09:33:23.613341+03
last_msg_receipt_time | 2024-11-07 09:33:23.613341+03
latest_end_lsn        | 
latest_end_time       | 2024-11-07 09:33:23.613341+03
-[ RECORD 2 ]---------+------------------------------
subid                 | 24708
subname               | test_sub
worker_type           | apply
pid                   | 5096
leader_pid            | 
relid                 | 
received_lsn          | C/1B800140
last_msg_send_time    | 2024-11-07 09:35:59.582918+03
last_msg_receipt_time | 2024-11-07 09:35:59.582973+03
latest_end_lsn        | C/1B800140
latest_end_time       | 2024-11-07 09:35:59.582918+03

В данный помент закончилась синхронизация данных, потребовалось 2 минуты 36 секунд.
```
