

#### Протестироват падение производителности при исползовании pgbouncer в разнх режимах: statement, transaction, session
    [databases]
    postgres = host=postgres17 port=5432 dbname=postgres

    [pgbouncer]
    listen_addr = *
    listen_port = 6432
    auth_type = plain
    auth_file = /etc/pgbouncer/userlist.txt
    pool_mode = session (меняем во время тестов)
    max_client_conn = 100
    default_pool_size = 20


    docker run --rm -it --network mynetwork postgres pgbench -T 60 -c 10 -j 2 -h pgbouncer -p 6432 -U postgres postgres
    session
        latency average = 0.989 ms
        initial connection time = 2.917 ms
        tps = 10110.033249 (without initial connection time)
    transaction
        latency average = 0.975 ms
        initial connection time = 3.489 ms
        tps = 10255.067469 (without initial connection time)

    statement
    -нельзя закидывать транзакции
        docker run --rm -it --network mynetwork postgres pgbench -T 60 -c 10 -j 2 -h pgbouncer -p 6432 -U postgres -S postgres
            latency average = 0.105 ms
            initial connection time = 3.795 ms
            tps = 95233.918819 (without initial connection time)

