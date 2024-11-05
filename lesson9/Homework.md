### Домашнее задание №8

1. Сгенерировать таблицу с 1 млн JSONB документов
```
CREATE TABLE test_jsonb AS
SELECT i AS id, (SELECT jsonb_object_agg(j, j) FROM generate_series(1, 1000) j) json
FROM generate_series(1, 10000) i;

Проверка размера TOAST
SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 'test_jsonb';
  heap_rel  | heap_rel_size |         toast_rel         | toast_rel_size 
------------+---------------+---------------------------+----------------
 test_jsonb | 50 MB         | pg_toast.pg_toast_1049180 | 2604 MB
```

2. Создать индекс

```
CREATE INDEX idx_json ON test_jsonb USING gin(json);
```

3. Обновить 1 из полей в json

```
UPDATE test_jsonb SET json = json::jsonb || '{"a":1}';

P.S. На диске во время обновления закончилось пространстово, поэтому не все строки обновлились
```


4. Убедиться в блоатинге TOAST

```
SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 'test_jsonb';
  heap_rel  | heap_rel_size |         toast_rel         | toast_rel_size 
------------+---------------+---------------------------+----------------
 test_jsonb | 88 MB         | pg_toast.pg_toast_1049180 | 4600 MB
```

5. Придумать методы избавится от него и проверить на практике

```
Самый простой и действенный метод это VACUUM FULL

postgres=# VACUUM FULL;
VACUUM
postgres=# SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 'test_jsonb';
  heap_rel  | heap_rel_size |         toast_rel         | toast_rel_size 
------------+---------------+---------------------------+----------------
 test_jsonb | 50 MB         | pg_toast.pg_toast_1049180 | 2604 MB
```


6. Не забываем про блоатинг индексов*

```
SELECT c.relname AS index_name, pg_size_pretty(pg_relation_size(c.oid)) AS index_size
FROM pg_class c
WHERE c.relname = 'idx_json';

 index_name | index_size 
------------+------------
 idx_json   | 1630 MB

REINDEX INDEX CONCURRENTLY public.idx_json;

 index_name | index_size 
------------+------------
 idx_json   | 819 MB
```

