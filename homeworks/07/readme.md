####  Развернуть ВМ (Linux) с PostgreSQL
    все как из первой домашки
####  Залить Тайские перевозки https://github.com/aeuge/postgres16book/tree/main/database
    не забыл vacuum analize;
####  Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)
                                                   ```->  Hash Join  (cost=43.40..2641.51 rows=144000 width=16) (actual time=0.242..22.011 rows=144000 loops=1)
                                                         Hash Cond: (r.fkschedule = s.id)
                                                         ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.006..5.577 rows=144000 loops=1)
                                                         ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.230..0.230 rows=1440 loops=1)
                                                               Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                                               ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.008..0.111 rows=1440 loops=1)
                                                   ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.016..0.016 rows=60 loops=1)
                                                         Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                         ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.005..0.009 rows=60 loops=1)
                                             ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.011..0.012 rows=10 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                                   ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.007..0.008 rows=10 loops=1)
                                       ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.079..0.089 rows=5 loops=1)
                                             Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                             ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.075..0.076 rows=5 loops=1)
                                                   Group Key: s_1.fkbus
                                                   Batches: 1  Memory Usage: 24kB
                                                   ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.024..0.034 rows=200 loops=1)
                           ->  Finalize GroupAggregate  (cost=257143.55..293326.69 rows=142819 width=12) (actual time=346.408..407.063 rows=144000 loops=1)
                                 Group Key: t.fkride
                                 ->  Gather Merge  (cost=257143.55..290470.31 rows=285638 width=12) (actual time=346.389..379.961 rows=431994 loops=1)
                                       Workers Planned: 2
                                       Workers Launched: 2
                                       ->  Sort  (cost=256143.53..256500.57 rows=142819 width=12) (actual time=336.979..343.162 rows=143998 loops=3)
                                             Sort Key: t.fkride
                                             Sort Method: external merge  Disk: 3672kB
                                             Worker 0:  Sort Method: external merge  Disk: 3672kB
                                             Worker 1:  Sort Method: external merge  Disk: 3672kB
                                             ->  Partial HashAggregate  (cost=218944.77..241472.48 rows=142819 width=12) (actual time=257.254..317.254 rows=143998 loops=3)
                                                   Group Key: t.fkride
                                                   Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27776kB
                                                   Worker 0:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27704kB
                                                   Worker 1:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27936kB
                                                   ->  Parallel Seq Scan on tickets t  (cost=0.00..80531.91 rows=2160591 width=12) (actual time=0.021..62.466 rows=1728502 loops=3)
 Planning Time: 0.857 ms
 JIT:
   Functions: 81
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 2.548 ms (Deform 0.917 ms), Inlining 0.000 ms, Optimization 2.084 ms, Emission 37.077 ms, Total 41.709 ms
 Execution Time: 578.360 ms
(61 rows)```

####  Навесить индексы на внешние ключ

   ``` CREATE INDEX idx_seat_fkbus ON book.seat(fkbus);
    CREATE INDEX idx_ride_fkschedule ON book.ride(fkschedule);
    CREATE INDEX idx_ride_fkbus ON book.ride(fkbus);
    CREATE INDEX idx_schedule_fkroute ON book.schedule(fkroute);
    CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute(fkbusstationfrom);
    CREATE INDEX idx_tickets_fkride ON book.tickets(fkride);```э

####  Проверить, помогли ли индексы на внешние ключи ускориться

                                                  ``` ->  Hash Join  (cost=43.40..2641.51 rows=144000 width=16) (actual time=0.338..25.125 rows=144000 loops=1)
                                                         Hash Cond: (r.fkschedule = s.id)
                                                         ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.004..6.050 rows=144000 loops=1)
                                                         ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.326..0.326 rows=1440 loops=1)
                                                               Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                                               ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.006..0.163 rows=1440 loops=1)
                                                   ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.020..0.021 rows=60 loops=1)
                                                         Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                         ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.005..0.011 rows=60 loops=1)
                                             ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.014..0.014 rows=10 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                                   ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.008..0.009 rows=10 loops=1)
                                       ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.091..0.092 rows=5 loops=1)
                                             Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                             ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.085..0.087 rows=5 loops=1)
                                                   Group Key: s_1.fkbus
                                                   Batches: 1  Memory Usage: 24kB
                                                   ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.028..0.040 rows=200 loops=1)
                           ->  Finalize GroupAggregate  (cost=257226.22..293574.54 rows=143471 width=12) (actual time=348.440..413.004 rows=144000 loops=1)
                                 Group Key: t.fkride
                                 ->  Gather Merge  (cost=257226.22..290705.12 rows=286942 width=12) (actual time=348.416..383.841 rows=431998 loops=1)
                                       Workers Planned: 2
                                       Workers Launched: 2
                                       ->  Sort  (cost=256226.19..256584.87 rows=143471 width=12) (actual time=337.044..343.537 rows=143999 loops=3)
                                             Sort Key: t.fkride
                                             Sort Method: external merge  Disk: 3672kB
                                             Worker 0:  Sort Method: external merge  Disk: 3672kB
                                             Worker 1:  Sort Method: external merge  Disk: 3672kB
                                             ->  Partial HashAggregate  (cost=218949.29..241484.12 rows=143471 width=12) (actual time=258.446..319.423 rows=143999 loops=3)
                                                   Group Key: t.fkride
                                                   Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 24016kB
                                                   Worker 0:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27472kB
                                                   Worker 1:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27512kB
                                                   ->  Parallel Seq Scan on tickets t  (cost=0.00..80532.52 rows=2160652 width=12) (actual time=0.016..66.096 rows=1728502 loops=3)
 Planning Time: 2.169 ms
 JIT:
   Functions: 81
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 2.961 ms (Deform 1.206 ms), Inlining 0.000 ms, Optimization 1.724 ms, Emission 32.415 ms, Total 37.101 ms
 Execution Time: 593.306 ms
(61 rows)```

    как итог индексы оказались бесполезны, таблички слишком маленькие как я понял, поэтому у нас везде происходит Seq Scan, а индексами мы только занимаем место
