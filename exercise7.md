1. Проверить скорость выполнения сложного запроса. Оценить затрачиваемые ресурсы с помощью команды EXPLAIN
2. Построить индексы на внешние ключи.
3. Ещё раз оценить тот же запрос с помощью команды EXPLAIN
4. Сделать выводы

```
-- Выполняем команду ANALYZE
thai=# analyze;
ANALYZE
```

```
-- Выполняем запрос с explain(analyze)
thai=# explain(analyze)
thai-# WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
```
План запроса:
```
 Limit  (cost=280167.30..280167.32 rows=10 width=56) (actual time=2705.655..2705.779 rows=10 loops=1)
   ->  Sort  (cost=280167.30..280528.51 rows=144485 width=56) (actual time=2705.653..2705.775 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 26kB
         ->  HashAggregate  (cost=272055.78..277045.03 rows=144485 width=56) (actual time=2631.037..2683.019 rows=144000 loops=1)
               Group Key: r.id, ((bs.city || ', '::text) || bs.name), (count(t.id)), (count(s_1.id))
               Planned Partitions: 32  Batches: 33  Memory Usage: 1049kB  Disk Usage: 14160kB
               ->  Hash Join  (cost=255071.46..264617.06 rows=144485 width=56) (actual time=2315.992..2570.798 rows=144000 loops=1)
                     Hash Cond: (r.fkbus = s_1.fkbus)
                     ->  Hash Join  (cost=255066.30..263188.72 rows=144485 width=36) (actual time=2315.862..2534.914 rows=144000 loops=1)
                           Hash Cond: (br.fkbusstationfrom = bs.id)
                           ->  Hash Join  (cost=255065.07..262647.49 rows=144485 width=24) (actual time=2315.839..2509.483 rows=144000 loops=1)
                                 Hash Cond: (s.fkroute = br.id)
                                 ->  Hash Join  (cost=255062.72..262239.08 rows=144485 width=24) (actual time=2315.805..2483.265 rows=144000 loops=1)
                                       Hash Cond: (r.fkschedule = s.id)
                                       ->  Hash Join  (cost=255019.32..261815.29 rows=144485 width=24) (actual time=2315.362..2455.695 rows=144000 loops=1)
                                             Hash Cond: (t.fkride = r.id)
                                             ->  Finalize HashAggregate  (cost=250296.32..253152.16 rows=144485 width=12) (actual time=2251.441..2322.010 rows=144000 loops=1)
                                                   Group Key: t.fkride
                                                   Planned Partitions: 16  Batches: 81  Memory Usage: 1081kB  Disk Usage: 15016kB
                                                   ->  Gather  (cost=199451.15..245132.11 rows=144485 width=12) (actual time=1702.025..2179.673 rows=288000 loops=1)
                                                         Workers Planned: 1
                                                         Workers Launched: 1
                                                         ->  Partial HashAggregate  (cost=198451.15..229683.61 rows=144485 width=12) (actual time=1699.759..2135.226 rows=144000 loops=2)
                                                               Group Key: t.fkride
                                                               Planned Partitions: 16  Batches: 81  Memory Usage: 1105kB  Disk Usage: 80528kB
                                                               Worker 0:  Batches: 81  Memory Usage: 1081kB  Disk Usage: 80592kB
                                                               ->  Parallel Seq Scan on tickets t  (cost=0.00..89428.51 rows=3050251 width=12) (actual time=0.029..936.089 rows=2592752 loops=2)
                                             ->  Hash  (cost=2219.00..2219.00 rows=144000 width=16) (actual time=63.672..63.673 rows=144000 loops=1)
                                                   Buckets: 32768  Batches: 16  Memory Usage: 686kB
                                                   ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.012..27.577 rows=144000 loops=1)
                                       ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.429..0.430 rows=1440 loops=1)
                                             Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                             ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.011..0.233 rows=1440 loops=1)
                                 ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.016..0.017 rows=60 loops=1)
                                       Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                       ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.003..0.008 rows=60 loops=1)
                           ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.009..0.010 rows=10 loops=1)
                                 Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                 ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.003..0.004 rows=10 loops=1)
                     ->  Hash  (cost=5.10..5.10 rows=5 width=12) (actual time=0.093..0.094 rows=5 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 9kB
                           ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.088..0.089 rows=5 loops=1)
                                 Group Key: s_1.fkbus
                                 Batches: 1  Memory Usage: 24kB
                                 ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.023..0.046 rows=200 loops=1)
 Planning Time: 3.236 ms
 Execution Time: 2721.187 ms
```

Строим индексы:
```
thai=# CREATE INDEX fkschedule_idx ON book.ride (fkschedule);
CREATE INDEX
thai=# CREATE INDEX fkroute_idx ON book.schedule (fkroute);
CREATE INDEX
thai=# CREATE INDEX fkbusstationfrom_idx ON book.busroute (fkbusstationfrom);
CREATE INDEX
thai=# CREATE INDEX fkride_idx ON book.tickets (fkride);
CREATE INDEX
thai=# CREATE INDEX fkbus_idx ON book.seat (fkbus);
CREATE INDEX
```

Выполняем запрос и смторим план:
```
 Limit  (cost=187832.29..187832.32 rows=10 width=56) (actual time=2291.838..2291.964 rows=10 loops=1)
   ->  Sort  (cost=187832.29..188193.51 rows=144485 width=56) (actual time=2291.837..2291.961 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 26kB
         ->  HashAggregate  (cost=179720.78..184710.02 rows=144485 width=56) (actual time=2217.840..2269.458 rows=144000 loops=1)
               Group Key: r.id, ((bs.city || ', '::text) || bs.name), (count(t.id)), (count(s_1.id))
               Planned Partitions: 32  Batches: 33  Memory Usage: 1049kB  Disk Usage: 14160kB
               ->  Hash Join  (cost=1053.00..172282.06 rows=144485 width=56) (actual time=5.469..2134.873 rows=144000 loops=1)
                     Hash Cond: (r.fkbus = s_1.fkbus)
                     ->  Hash Join  (cost=1047.84..170853.72 rows=144485 width=36) (actual time=5.387..2079.791 rows=144000 loops=1)
                           Hash Cond: (br.fkbusstationfrom = bs.id)
                           ->  Hash Join  (cost=1046.61..170312.48 rows=144485 width=24) (actual time=5.372..2047.986 rows=144000 loops=1)
                                 Hash Cond: (s.fkroute = br.id)
                                 ->  Hash Join  (cost=1044.26..169904.07 rows=144485 width=24) (actual time=5.346..2016.334 rows=144000 loops=1)
                                       Hash Cond: (r.fkschedule = s.id)
                                       ->  Merge Join  (cost=1000.86..169480.29 rows=144485 width=24) (actual time=4.820..1973.652 rows=144000 loops=1)
                                             Merge Cond: (r.id = t.fkride)
                                             ->  Index Scan using ride_pkey on ride r  (cost=0.42..3377.32 rows=144000 width=16) (actual time=0.035..35.004 rows=144000 loops=1)
                                             ->  Finalize GroupAggregate  (cost=1000.44..162492.05 rows=144485 width=12) (actual time=4.780..1894.072 rows=144000 loops=1)
                                                   Group Key: t.fkride
                                                   ->  Gather Merge  (cost=1000.44..160324.78 rows=144485 width=12) (actual time=4.719..1847.270 rows=144000 loops=1)
                                                         Workers Planned: 1
                                                         Workers Launched: 1
                                                         ->  Partial GroupAggregate  (cost=0.43..143070.21 rows=144485 width=12) (actual time=0.100..1613.316 rows=72000 loops=2)
                                                               Group Key: t.fkride
                                                               ->  Parallel Index Scan using fkride_idx on tickets t  (cost=0.43..126373.87 rows=3050297 width=12) (actual time=0.032..1352.777 rows=2592752 loops=2)
                                       ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.518..0.519 rows=1440 loops=1)
                                             Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                             ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.011..0.271 rows=1440 loops=1)
                                 ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.022..0.023 rows=60 loops=1)
                                       Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                       ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.004..0.011 rows=60 loops=1)
                           ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.009..0.010 rows=10 loops=1)
                                 Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                 ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.004..0.006 rows=10 loops=1)
                     ->  Hash  (cost=5.10..5.10 rows=5 width=12) (actual time=0.074..0.075 rows=5 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 9kB
                           ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.070..0.072 rows=5 loops=1)
                                 Group Key: s_1.fkbus
                                 Batches: 1  Memory Usage: 24kB
                                 ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.019..0.035 rows=200 loops=1)
 Planning Time: 1.306 ms
 Execution Time: 2294.571 ms
```

По времени выполнения запрос с индексами отработал на 16% быстрее (2721мс -> 2294мс). А также сократилась стоимость запроса (280167 -> 187832)
Если бы в таблицах было бы записей в несколько раз больше, то я полагаю, что мы бы смогли увидеть больший прирост производительности.


