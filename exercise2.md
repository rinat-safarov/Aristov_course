-- сделать в первой сессии новую таблицу и наполнить ее данными
```
BEGIN;
CREATE TABLE persons(id serial, first_name text, second_name text);
INSERT INTO persons(first_name, second_name) VALUES ('dmitry',
'dmitryev');
INSERT INTO persons(first_name, second_name) VALUES ('anna', 'ievleva');
COMMIT;
BEGIN
CREATE TABLE
INSERT 0 1
INSERT 0 1
COMMIT
```

READ COMMITTED
--------------


-- посмотреть текущий уровень изоляции: 
```
thai=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 строка)

thai=# BEGIN;
-- в первой сессии добавить новую запись

INSERT INTO persons(first_name, second_name) VALUES ('elena',
'koroleva');
BEGIN
INSERT 0 1
```
-- сделать запрос на выбор всех записей во второй сессии

`SELECT * FROM persons;`

Во второй сессии видим только первые две записи:
```
thai=# BEGIN;
BEGIN
thai=*# SELECT * FROM persons;
 id | first_name | second_name
----+------------+-------------
  1 | dmitry     | dmitryev
  2 | anna       | ievleva
(2 строки)
```

-- завершить транзакцию в первом окне

```
COMMIT;
thai=*# COMMIT;
COMMIT
```

сделать запрос во выбор всех записей второй сессии

```
thai=*# SELECT * FROM persons;
 id | first_name | second_name
----+------------+-------------
  1 | dmitry     | dmitryev
  2 | anna       | ievleva
  3 | elena      | koroleva
(3 строки)
```
Запись видно, потому что мы завершили транзакцию в первой сессии и на
уровне read committed возможно неповторяющееся чтение

REPEATABLE READ
---------------
```
thai=# BEGIN transaction isolation level repeatable read;
BEGIN
thai=*# INSERT INTO persons(first_name, second_name) VALUES ('polina',
'tsvetkova');
INSERT 0 1
```
-- сделать запрос на выбор всех записей во второй сессии
```
thai=# BEGIN transaction isolation level repeatable read;
BEGIN
thai=*# SELECT * FROM persons;
 id | first_name | second_name
----+------------+-------------
  1 | dmitry     | dmitryev
  2 | anna       | ievleva
  3 | elena      | koroleva
(3 строки)
```
-- !!! Нет, запись не видно. Грязное чтение не допускается на уровне

изоляции repeatable read !!!


-- завершить транзакцию в первом окне

`COMMIT;`

-- сделать запрос во выбор всех записей второй сессии

```
thai=*# SELECT * FROM persons;
 id | first_name | second_name
----+------------+-------------
  1 | dmitry     | dmitryev
  2 | anna       | ievleva
  3 | elena      | koroleva
(3 строки)
```
-- !!! Нет, запись не видно. Неповторяющееся чтение не допускается на
уровне изоляции repeatable read в Постгресе!!!
