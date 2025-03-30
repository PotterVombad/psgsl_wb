#### Создать таблицу accounts(id integer, amount numeric);
#### Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
    первый терминал: 
        begin;
        update accounts set amount = amount + 2 where id = 1;
        update accounts set amount = amount + 2 where id = 3;
        select * from accounts;
            ERROR:  deadlock detected
            DETAIL:  Process 2261 waits for ShareLock on transaction 958; blocked by process 2282.
            Process 2282 waits for ShareLock on transaction 957; blocked by process 2261.
            HINT:  See server log for query details.
            CONTEXT:  while updating tuple (0,3) in relation "accounts"
        первая транзакция соответственно сломалась
            ERROR:  current transaction is aborted, commands ignored until end of transaction block
        rollback;
    второй терминал:
        begin
        update accounts set amount = amount + 2 where id = 3;
        update accounts set amount = amount + 2 where id = 1;
        commit;

#### Посмотреть логи и убедиться, что информация о дедлоке туда попала.
    сохранение логов в файлы выключено, но в принципе его вывело выше
        SHOW logging_collector;
        logging_collector 
        -------------------
        off
        (1 row)