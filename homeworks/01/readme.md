#### Создал докер контейнер с 17 постгресом
```
docker run --name postgres17 \
  -e POSTGRES_PASSWORD=123 \
  -p 5432:5432 \
  -d postgres:17
```

#### Скачал, распаковал и закинул бд в постгрес 

`wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && docker exec -i postgres17 psql -U postgres < thai.sql`

#### Зашел в постгрес в нужную бд и выполнил запрос

`docker exec -it postgres17 psql -U postgres -d thai`

`select count(*) from book.tickets;`

#### Получил ответ

```
count  
---------
 5185505
(1 row)
```