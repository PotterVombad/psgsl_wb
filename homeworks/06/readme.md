####  Развернуть асинхронную реплику (можно использовать 1 ВМ, просто рядом кластер развернуть и подключиться через localhost):
        тестируем производительность по сравнению с сингл инстансом
    
    очень долго и муторно ставилась реплика в докер контейнере, но все таки на второ1 день что то получлось. я залил тайские перевозки на мастера и увидел на реплике:
        docker exec -it postgres_replica psql -U postgres -c "\l"
                                                                List of databases
              Name    |  Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | Locale | ICU Rules |   Access privileges   
            ----------+----------+----------+-----------------+------------+------------+--------+-----------+-----------------------
            postgres  | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
            template0 | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
                      |          |          |                 |            |            |        |           | postgres=CTc/postgres
            template1 | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
                      |          |          |                 |            |            |        |           | postgres=CTc/postgres
            thai      | postgres | UTF8     | libc            | C.UTF-8    | C.UTF-8    |        |           | 
            (4 rows)
    
    тестирование с включеннной репликацией:
    docker exec -it postgres_master pgbench -U postgres -c 10 -T 30 postgres
        latency average = 0.914 ms
        initial connection time = 9.492 ms
        tps = 10935.325540 (without initial connection time)
    с выключенной:
        latency average = 0.667 ms
        initial connection time = 9.018 ms
        tps = 14985.798306 (without initial connection time)