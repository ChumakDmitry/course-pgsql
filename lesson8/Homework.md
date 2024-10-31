### Домашнее задание №7

1. Создать таблицу с продажами.

```
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
);

DO $$
DECLARE
    start_date DATE = '2023-01-01';
    end_date DATE = '2023-12-31';
    sale_date DATE;
    amount DECIMAL(10, 2);
    i INT = 1;
BEGIN
        WHILE i <= 100 LOOP
            sale_date = start_date + ((end_date - start_date) * RANDOM())::INT;
            amount = ROUND((RANDOM() * 1000)::DECIMAL, 2);
            INSERT INTO sales (sale_date, amount) VALUES (sale_date, amount);
            i = i + 1;
        END LOOP;
END $$;
```


2. Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд)

а. через case предусмотреть NULL на входе
```
CREATE OR REPLACE FUNCTION get_quater2(sale_date DATE) RETURNS TEXT AS $$
BEGIN
    IF sale_date IS NULL THEN RETURN NULL; END IF;
    RETURN CASE
        WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 1 AND 4 THEN 'Первая треть'
        WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 5 AND 8 THEN 'Вторая треть'
        WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 9 AND 12 THEN 'Третья треть'
        ELSE 'Неверная дата'
    END;
END;
$$ LANGUAGE plpgsql RETURNS NULL ON NULL INPUT;
```
b. * (бонуса в виде зачета дз не будет) используя математическую операцию (лучше 2+ варианта)

4. Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё отработало

```
SELECT id, sale_date, get_quater(sale_date) FROM sales;

 id  | sale_date  |  get_quater  
-----+------------+--------------
   1 | 2023-06-26 | Вторая треть
   2 | 2023-09-06 | Третья треть
   3 | 2023-11-17 | Третья треть
   4 | 2023-04-25 | Первая треть
   5 | 2023-03-08 | Первая треть
   6 | 2023-04-13 | Первая треть
   7 | 2023-12-06 | Третья треть
   8 | 2023-06-29 | Вторая треть
   9 | 2023-12-21 | Третья треть
  10 | 2023-09-26 | Третья треть
```
