### Домашнее задание №6

1. Развернуть ВМ (Linux) с PostgreSQL

С помощью Oracle VM VirtualBox развернул Ubuntu 22.04
2. Залить Тайские перевозки

https://github.com/aeuge/postgres16book/tree/main/database
``wget https://storage.googleapis.com/thaibus/thai_medium.tar.gz && tar -xf thai_medium.tar.gz && psql < thai.sql``

3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)

Проверим скорость с помощью команды Explain analyze. Увидим что стоимость нашего запроса составляет `cost=3475183.60..3475183.62` 'бананов'
```
Limit  (cost=3475183.60..3475183.62 rows=10 width=56) (actual time=131396.780..131396.957 rows=10 loops=1)
   ->  Sort  (cost=3475183.60..3479037.25 rows=1541460 width=56) (actual time=131039.707..131039.868 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 26kB
         ->  HashAggregate  (cost=3388644.66..3441873.20 rows=1541460 width=56) (actual time=127886.771..129585.533 rows=1500000 loops=1)
               Group Key: r.id, ((bs.city || ', '::text) || bs.name), (count(t.id)), (count(s_1.id))
               Planned Partitions: 32  Batches: 33  Memory Usage: 8209kB  Disk Usage: 96144kB
               ->  Hash Join  (cost=2701079.18..3221974.30 rows=1541460 width=56) (actual time=89928.292..125558.987 rows=1500000 loops=1)
                     Hash Cond: (r.fkbus = s_1.fkbus)
                     ->  Nested Loop  (cost=2701074.07..3206785.80 rows=1541460 width=84) (actual time=89925.352..122389.667 rows=1500000 loops=1)
                           ->  Hash Join  (cost=2701073.92..3170137.40 rows=1541460 width=24) (actual time=89922.316..116373.500 rows=1500000 loops=1)
                                 Hash Cond: (s.fkroute = br.id)
                                 ->  Hash Join  (cost=2701071.57..3165802.91 rows=1541460 width=24) (actual time=89922.142..113414.661 rows=1500000 loops=1)
                                       Hash Cond: (r.fkschedule = s.id)
                                       ->  Merge Join  (cost=2701025.82..3161699.39 rows=1541460 width=24) (actual time=89918.288..110417.430 rows=1500000 loops=1)
                                             Merge Cond: (r.id = t.fkride)
                                             ->  Index Scan using ride_pkey on ride r  (cost=0.43..47127.43 rows=1500000 width=16) (actual time=1.271..2004.774 rows=1500000 loops=1)
                                             ->  Finalize GroupAggregate  (cost=2701025.40..3091553.71 rows=1541460 width=12) (actual time=89916.984..103904.315 rows=1500000 loops=1)
                                                   Group Key: t.fkride
                                                   ->  Gather Merge  (cost=2701025.40..3060724.51 rows=3082920 width=12) (actual time=89916.886..97799.471 rows=4499999 loops=1)
                                                         Workers Planned: 2
                                                         Workers Launched: 2
                                                         ->  Sort  (cost=2700025.37..2703879.02 rows=1541460 width=12) (actual time=89844.815..91848.965 rows=1500000 loops=3)
                                                               Sort Key: t.fkride
                                                               Sort Method: external merge  Disk: 38232kB
                                                               Worker 0:  Sort Method: external merge  Disk: 38232kB
                                                               Worker 1:  Sort Method: external merge  Disk: 38232kB
                                                               ->  Partial HashAggregate  (cost=2280104.91..2515250.65 rows=1541460 width=12) (actual time=80564.866..87462.753 rows=1500000 loops=3)
                                                                     Group Key: t.fkride
                                                                     Planned Partitions: 32  Batches: 33  Memory Usage: 8209kB  Disk Usage: 553992kB
                                                                     Worker 0:  Batches: 33  Memory Usage: 8209kB  Disk Usage: 554448kB
                                                                     Worker 1:  Batches: 33  Memory Usage: 8209kB  Disk Usage: 553040kB
                                                                     ->  Parallel Seq Scan on tickets t  (cost=0.00..838668.68 rows=22500468 width=12) (actual time=1.186..39399.062 rows=17999158 loops=3)
                                       ->  Hash  (cost=27.00..27.00 rows=1500 width=8) (actual time=3.824..3.827 rows=1500 loops=1)
                                             Buckets: 2048  Batches: 1  Memory Usage: 75kB
                                             ->  Seq Scan on schedule s  (cost=0.00..27.00 rows=1500 width=8) (actual time=0.535..2.240 rows=1500 loops=1)
                                 ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.144..0.146 rows=60 loops=1)
                                       Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                       ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.010..0.074 rows=60 loops=1)
                           ->  Memoize  (cost=0.15..0.36 rows=1 width=68) (actual time=0.001..0.001 rows=1 loops=1500000)
                                 Cache Key: br.fkbusstationfrom
                                 Cache Mode: logical
                                 Hits: 1499990  Misses: 10  Evictions: 0  Overflows: 0  Memory Usage: 2kB
                                 ->  Index Scan using busstation_pkey on busstation bs  (cost=0.14..0.35 rows=1 width=68) (actual time=0.300..0.301 rows=1 loops=10)
                                       Index Cond: (id = br.fkbusstationfrom)
                     ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=2.912..2.917 rows=5 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 9kB
                           ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=2.895..2.903 rows=5 loops=1)
                                 Group Key: s_1.fkbus
                                 Batches: 1  Memory Usage: 24kB
                                 ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=2.430..2.626 rows=200 loops=1)
 Planning Time: 18.421 ms
 JIT:
   Functions: 88
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 6.835 ms, Inlining 337.001 ms, Optimization 305.574 ms, Emission 199.005 ms, Total 848.415 ms
 Execution Time: 131542.111 ms
(57 rows)
```
4. Навесить индексы на внешние ключ

Исходя из модели, приведённой в github, добавим индексы на внешние ключи
```
CREATE INDEX idx_seat_fkbus ON book.seat (fkbus);
CREATE INDEX idx_tickets_fkride ON book.tickets (fkride);
CREATE INDEX idx_ride_fkschedule ON book.ride (fkschedule);
CREATE INDEX idx_schedule_fkroute ON book.schedule (fkroute);
CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute (fkbusstationfrom);
```
5. Проверить, помогли ли индексы на внешние ключи ускориться

Нет, не помогли, т.к. группировка таблицы tickets и сортировка забирают большую часть ресурсов.
Попробовал добавить индекс на сортировку `CREATE INDEX idx_ride_startdate ON book.ride (startdate);`
Отключение seqscan и vacuum analyze тоже не помогли

Решение которое помогло

Создаём мат вью запросом 
```
CREATE MATERIALIZED VIEW mv_order_all_place AS
SELECT fkride, count(id) AS order_place
FROM book.tickets
GROUP BY fkride;
```

Накидываем индекс на поле `fkride`: `create index idx_mv_fkride on mv_order_all_place(fkride);` (не бейте за название индекса, сидел долго и уже устал названия длинные писать)

И вот наш результат
```
 Limit  (cost=239636.06..239636.09 rows=10 width=56) (actual time=24954.601..24954.661 rows=10 loops=1)
   ->  Sort  (cost=239636.06..243386.06 rows=1500000 width=56) (actual time=24929.035..24929.078 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 26kB
         ->  HashAggregate  (cost=184721.60..207221.60 rows=1500000 width=56) (actual time=21922.995..23475.599 rows=1500000 loops=1)
               Group Key: r.id, ((bs.city || ', '::text) || bs.name), t.order_place, (count(s_1.id))
               Batches: 1  Memory Usage: 229393kB
               ->  Hash Join  (cost=134.26..169721.60 rows=1500000 width=56) (actual time=4.676..19676.382 rows=1500000 loops=1)
                     Hash Cond: (r.fkbus = s_1.fkbus)
                     ->  Hash Join  (cost=117.00..154929.34 rows=1500000 width=36) (actual time=4.187..16725.124 rows=1500000 loops=1)
                           Hash Cond: (br.fkbusstationfrom = bs.id)
                           ->  Hash Join  (cost=104.59..149310.68 rows=1500000 width=24) (actual time=4.130..13983.250 rows=1500000 loops=1)
                                 Hash Cond: (s.fkroute = br.id)
                                 ->  Hash Join  (cost=90.80..145081.27 rows=1500000 width=24) (actual time=3.263..11239.966 rows=1500000 loops=1)
                                       Hash Cond: (r.fkschedule = s.id)
                                       ->  Merge Join  (cost=6.27..141048.12 rows=1500000 width=24) (actual time=0.042..8439.296 rows=1500000 loops=1)
                                             Merge Cond: (r.id = t.fkride)
                                             ->  Index Scan using ride_pkey on ride r  (cost=0.43..47076.43 rows=1500000 width=16) (actual time=0.011..1657.217 rows=1500000 loops=1)
                                             ->  Index Scan using idx_mv_fkride on mv_order_all_place t  (cost=0.43..71472.33 rows=1500000 width=12) (actual time=0.010..2351.500 rows=1500000 loops=1)
                                       ->  Hash  (cost=65.78..65.78 rows=1500 width=8) (actual time=3.197..3.200 rows=1500 loops=1)
                                             Buckets: 2048  Batches: 1  Memory Usage: 75kB
                                             ->  Index Scan using schedule_pkey on schedule s  (cost=0.28..65.78 rows=1500 width=8) (actual time=0.012..1.638 rows=1500 loops=1)
                                 ->  Hash  (cost=13.04..13.04 rows=60 width=8) (actual time=0.849..0.852 rows=60 loops=1)
                                       Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                       ->  Index Scan using idx_busroute_fkbusstationfrom on busroute br  (cost=0.14..13.04 rows=60 width=8) (actual time=0.710..0.782 rows=60 loops=1)
                           ->  Hash  (cost=12.29..12.29 rows=10 width=20) (actual time=0.038..0.041 rows=10 loops=1)
                                 Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                 ->  Index Scan using busstation_pkey on busstation bs  (cost=0.14..12.29 rows=10 width=20) (actual time=0.011..0.022 rows=10 loops=1)
                     ->  Hash  (cost=17.20..17.20 rows=5 width=12) (actual time=0.465..0.469 rows=5 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 9kB
                           ->  GroupAggregate  (cost=0.14..17.20 rows=5 width=12) (actual time=0.135..0.454 rows=5 loops=1)
                                 Group Key: s_1.fkbus
                                 ->  Index Scan using idx_seat_fkbus on seat s_1  (cost=0.14..16.14 rows=200 width=8) (actual time=0.034..0.242 rows=200 loops=1)
 Planning Time: 3.090 ms
 JIT:
   Functions: 48
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 1.954 ms, Inlining 0.000 ms, Optimization 0.989 ms, Emission 24.702 ms, Total 27.646 ms
 Execution Time: 24986.789 ms
(39 rows)
```
