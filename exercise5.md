При использовании запросов в pgbench самый лучший результат показал режим transaction

```
Создадим в конфиге баунсера 2 алиаса:

Запустим тест в режиме "session":
---------------------------------
pgbench thaiS -p 6432 -S
pgbench (14.13 (Debian 14.13-1.pgdg110+1))
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 10/10
latency average = 0.118 ms
initial connection time = 0.229 ms
tps = 4474.576271 (without initial connection time)

Теперь запустим тест в режиме "transaction":
--------------------------------------------
pgbench thaiT -p 6432 -S
pgbench (14.13 (Debian 14.13-1.pgdg110+1))
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 10/10
latency average = 0.212 ms
initial connection time = 0.301 ms
tps = 8710.315591 (without initial connection time)

```
Конец задания
