Создаём таблицу:
```
create table accounts(id integer, amount numeric);
CREATE TABLE
```

Добавляем строки:
```
INSERT INTO accounts VALUES (1,2000.00), (2,2000.00), (3,2000.00);
INSERT 0 3
```

В первой сессии открываем транзакцию и делаем UPDATE:
```
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 1;
BEGIN
UPDATE 1
```

Во второй сессии также открываем транзакцию и делвем UPDATE:
```
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 2;
BEGIN
UPDATE 1
```

В первой сессии делаем ещё один UPDATE:
```
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 1;
BEGIN
UPDATE 1
```

И во второй сессии делаем ещё один UPDATE и получаем deadlock:
```
UPDATE accounts SET amount = amount + 1 WHERE id = 1;
ERROR:  deadlock detected
ПОДРОБНОСТИ:  Process 1641459 waits for ShareLock on transaction 1447; blocked by process 1640539.
Process 1640539 waits for ShareLock on transaction 1448; blocked by process 1641459.
ПОДСКАЗКА:  See server log for query details.
КОНТЕКСТ:  while updating tuple (0,1) in relation "accounts"
```


