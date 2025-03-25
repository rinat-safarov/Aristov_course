>1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
  * **_sudo -u postgres psql_**
    * create table test_table_rinat(c1 text);
    * INSERT INTO test_table_rinat(c1) SELECT 'noname' FROM generate_series(1,1000000);
```
thai=# create table test_table_rinat(c1 text);
CREATE TABLE

thai=# INSERT INTO test_table_rinat(c1) SELECT 'foo' FROM generate_series(1,1000000);
INSERT 0 1000000
```

>2. Посмотреть размер файла с таблицей
 * **_select pg_size_pretty(pg_table_size('test_table_rinat'));_**

```
thai=# select pg_size_pretty(pg_table_size('test_table_rinat'));
 pg_size_pretty
----------------
 35 MB
(1 строка)
```

> 3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
 * update test_table_rinat set c1=c1||'*';
```
update test_table_rinat set c1=c1||'!';
UPDATE 1000000
update test_table_rinat set c1=c1||'@';
UPDATE 1000000
update test_table_rinat set c1=c1||'#';
UPDATE 1000000
update test_table_rinat set c1=c1||'$';
UPDATE 1000000
update test_table_rinat set c1=c1||'%';
UPDATE 1000000
```
> 4 Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
 * **_SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test_table_rinat';_**
   * Мертвых строчек n_dead_tup нет т.к. avtovacuum быстрее нас :-) Но должно было быть примерно 5 млн. мертвых строк.
   * Автовакуум пришел в "2025-03-25 18:03:04.02744+03"
```
     relname      | n_live_tup | n_dead_tup | ratio% |       last_autovacuum
------------------+------------+------------+--------+------------------------------
 test_table_rinat |    1000000 |          0 |      0 | 2025-03-25 18:03:04.02744+03
(1 строка)
```

> 5 раз обновить все строчки и добавить к каждой строчке любой символ
 * update test_table_rinat set c1=c1||'9';
```
update test_table_rinat set c1=c1||'6';
UPDATE 1000000
Время: 1829,690 мс (00:01,830)
update test_table_rinat set c1=c1||'7';
UPDATE 1000000
Время: 1994,549 мс (00:01,995)
update test_table_rinat set c1=c1||'8';
UPDATE 1000000
Время: 2019,311 мс (00:02,019)
thai=# update test_table_rinat set c1=c1||'9';
UPDATE 1000000
Время: 1753,774 мс (00:01,754)
thai=# update test_table_rinat set c1=c1||'0';
UPDATE 1000000
Время: 1770,336 мс (00:01,770)
```
> Посмотреть размер файла с таблицей
 * **_select pg_size_pretty(pg_table_size('test_table_rinat'));_**
```
select pg_size_pretty(pg_table_size('test_table_rinat'));
 pg_size_pretty
----------------
 85 MB
(1 строка)
```
> Отключить Автовакуум на конкретной таблице
 * **_alter table test_table_rinat set (autovacuum_enabled = off);_**
```
alter table test_table_rinat set (autovacuum_enabled = off);
ALTER TABLE
```

> 10 раз обновить все строчки и добавить к каждой строчке любой символ
 * update test_table_rinat set c1=c1||'1';
```
thai=# update test_table_rinat set c1=c1||'1';
UPDATE 1000000
Время: 1828,932 мс (00:01,829)
thai=# update test_table_rinat set c1=c1||'2';
UPDATE 1000000
Время: 2014,068 мс (00:02,014)
thai=# update test_table_rinat set c1=c1||'3';
UPDATE 1000000
Время: 2399,917 мс (00:02,400)
thai=# update test_table_rinat set c1=c1||'4';
UPDATE 1000000
Время: 2210,401 мс (00:02,210)
thai=# update test_table_rinat set c1=c1||'5';
UPDATE 1000000
Время: 2208,813 мс (00:02,209)
thai=# update test_table_rinat set c1=c1||'6';
UPDATE 1000000
Время: 1947,053 мс (00:01,947)
thai=# update test_table_rinat set c1=c1||'7';
UPDATE 1000000
Время: 2146,278 мс (00:02,146)
thai=# update test_table_rinat set c1=c1||'8';
UPDATE 1000000
Время: 1930,776 мс (00:01,931)
thai=# update test_table_rinat set c1=c1||'9';
UPDATE 1000000
Время: 2128,117 мс (00:02,128)
thai=# update test_table_rinat set c1=c1||'0';
UPDATE 1000000
Время: 2820,057 мс (00:02,820)
```

> Посмотреть размер файла с таблицей
 * **_select pg_size_pretty(pg_table_size('test_table_rinat'));_**
```
select pg_size_pretty(pg_table_size('test_table_rinat'));
 pg_size_pretty
----------------
 525 MB
(1 строка)
```

> 4 Посмотреть количество мертвых строчек в таблице
 * **_SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test_table_rinat';_**

```
     relname      | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
------------------+------------+------------+--------+-------------------------------
 test_table_rinat |    1000000 |    9997540 |    999 | 2025-03-25 18:06:56.386783+03
(1 строка)
```

> Объясните полученный результат
 * **_Автовакуум выключен и мертвые строчки не удаляются. В итоге, у нас 9997540. мертвых строк. Но чтобы сжать файл на уровне ОС-надо выполнить vacuum full_**

> Не забудьте включить автовакуум)
 * **_alter table test_table_rinat set (autovacuum_enabled = on);_**

> Задание со *:
> Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
> Не забыть вывести номер шага цикла.
> 
 * _DO_
 * _$do$_
 * _BEGIN_
   * _FOR i IN 1..10 LOOP_
     * _RAISE NOTICE 'Step = %', i;_
     * _update test_table_rinat set c1=c1||i;_
   * _END LOOP;_
 * _END_
 * _$do$;_

```
NOTICE:  Step = 1
NOTICE:  Step = 2
NOTICE:  Step = 3
NOTICE:  Step = 4
NOTICE:  Step = 5
NOTICE:  Step = 6
NOTICE:  Step = 7
NOTICE:  Step = 8
NOTICE:  Step = 9
NOTICE:  Step = 10
DO
Время: 21457,875 мс (00:21,458)
```

