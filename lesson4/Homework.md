### Домашняя работа №4

1. Создать таблицу accounts(id integer, amount numeric);
```
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    amount NUMERIC
);
```


2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
```
INSERT INTO accounts (amount) VALUES (100), (200), (300);
```

Для получения состояния дедлока будем использовать операцию update в паралелльных транзакциях.
Т1 - терминал №1, Т2 - терминал №2

```
--T1
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 1;
```

```
--T2
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 2;
```

```
--T1
UPDATE accounts SET amount = amount + 1 WHERE id = 2;
```

```
--T2
UPDATE accounts SET amount = amount + 1 WHERE id = 1;
```
В результате выполнения операции в первой консоли выполнились, а операция во второй консоли ушли в ошибку
```
--T2
ERROR:  deadlock detected
DETAIL:  Process 6668 waits for ShareLock on transaction 878; blocked by process 6705.
Process 6705 waits for ShareLock on transaction 879; blocked by process 6668.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,8) in relation "accounts"

```

3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.
![image](https://github.com/user-attachments/assets/2432bdc5-08b6-4b5b-b581-399f955d73e0)


Произошло это из-за того, что мы пытались обновить одни и те же строки в разных порядках в рамках одной транзакции. Из-за того, что блокировка типа `ShareLock` в одной транзакиции не могла снятся, пока не выполнится другая транзакция


