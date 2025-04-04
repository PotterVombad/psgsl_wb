## Развернуть ВМ (Linux) с PostgreSQL
Все как из первой домашки.

---

## Залить "Тайские перевозки"  
[Источник данных на GitHub](https://github.com/aeuge/postgres16book/tree/main/database)  

Не забыть выполнить:
```sql
VACUUM ANALYZE;
```

---

## Проверить скорость выполнения сложного запроса (до индексов)

```
->  Hash Join  (cost=43.40..2641.51 rows=144000 width=16) (actual time=0.242..22.011 rows=144000 loops=1)
      Hash Cond: (r.fkschedule = s.id)
      ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.006..5.577 rows=144000 loops=1)
      ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.230..0.230 rows=1440 loops=1)
            Buckets: 2048  Batches: 1  Memory Usage: 73kB
            ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.008..0.111 rows=1440 loops=1)
...
Execution Time: 578.360 ms
```

---

## Навесить индексы на внешние ключи

```sql
CREATE INDEX idx_seat_fkbus ON book.seat(fkbus);
CREATE INDEX idx_ride_fkschedule ON book.ride(fkschedule);
CREATE INDEX idx_ride_fkbus ON book.ride(fkbus);
CREATE INDEX idx_schedule_fkroute ON book.schedule(fkroute);
CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute(fkbusstationfrom);
CREATE INDEX idx_tickets_fkride ON book.tickets(fkride);
```

---

## Проверить, помогли ли индексы

```
->  Hash Join  (cost=43.40..2641.51 rows=144000 width=16) (actual time=0.338..25.125 rows=144000 loops=1)
      Hash Cond: (r.fkschedule = s.id)
      ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.004..6.050 rows=144000 loops=1)
      ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.326..0.326 rows=1440 loops=1)
            Buckets: 2048  Batches: 1  Memory Usage: 73kB
            ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.006..0.163 rows=1440 loops=1)
...
Execution Time: 593.306 ms
```

---

## Вывод

Индексы оказались бесполезны — таблички слишком маленькие, поэтому везде происходят `Seq Scan`, а индексы только занимают место.
