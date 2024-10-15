### Домашнее задание №3

1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данными в размере 1 млн строк
```
CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    text_field TEXT
);

INSERT INTO test_table (text_field)
SELECT md5(random()::text)
FROM generate_series(1, 1000000);
```

2. Посмотреть размер файла с таблицей
```
SELECT pg_size_pretty(pg_total_relation_size('test_table'));

 pg_size_pretty 
----------------
 87 MB
```

### 3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```
DO $$
BEGIN
    FOR i IN 1..5 LOOP
        UPDATE test_table SET text_field = text_field || 'a';
    END LOOP;
END $$;
```

4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```
SELECT relname, n_dead_tup FROM pg_stat_user_tables WHERE relname = 'test_table';

  relname   | n_dead_tup 
------------+------------
 test_table |    5000000

SELECT last_autovacuum FROM pg_stat_user_tables WHERE relname = 'test_table';

        last_autovacuum        
-------------------------------
 2024-10-15 14:05:13.637217+03
```

5. Подождать некоторое время, проверяя, пришел ли автовакуум
```
SELECT last_autovacuum FROM pg_stat_user_tables WHERE relname = 'test_table';
        last_autovacuum        
-------------------------------
 2024-10-15 14:07:41.093645+03

```

6. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```
DO $$
BEGIN
    FOR i IN 1..5 LOOP
        UPDATE test_table SET text_field = text_field || 'b';
    END LOOP;
END $$;
```

7. Посмотреть размер файла с таблицей
```
SELECT pg_size_pretty(pg_total_relation_size('test_table'));

pg_size_pretty 
----------------
 524 MB
```

8. Отключить Автовакуум на конкретной таблице
```
ALTER TABLE test_table SET (autovacuum_enabled = false);
```

9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
```
DO $$
BEGIN
    FOR i IN 1..10 LOOP
        UPDATE test_table SET text_field = text_field || 'c';
    END LOOP;
END $$;
```

10. Посмотреть размер файла с таблицей
```
SELECT pg_size_pretty(pg_total_relation_size('test_table'));
```

### 11. Объясните полученный результат

- Размер файла после первых 5 обновлений: После первых 5 обновлений размер файла увеличился, так как данные были изменены. После прихода автовакуума размер не изменился, т.к. он пометил строки готовыми для записи

- Размер файла после вторых 5 обновлений: Размер файла слегка увеличился, т.к. новые данные записались на место старых(мёртвых) строк.

- Размер файла после отключения автовакуума и 10 обновлений: Размер файла значительно увеличился, так как автовакуум был отключен, и все старые версии строк остались в таблице без очистки.

12. Не забудьте включить автовакуум
```
ALTER TABLE test_table SET (autovacuum_enabled = true);
```
