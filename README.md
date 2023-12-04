# Home Work otus-pgsql-hw-lesson-3

зайти вторым ssh (вторая сессия)
запустить везде psql из под пользователя postgres

done


выключить auto commit

    \set AUTOCOMMIT off

done

сделать:

##  1.	в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

  postgres=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
  CREATE TABLE
  INSERT 0 1
  INSERT 0 1
  COMMIT
  postgres=#


  postgres=*# select * from persons;
   id | first_name | second_name
  ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
  (2 rows)



##  2.	посмотреть текущий уровень изоляции: show transaction isolation level

    postgres=# show transaction isolation level;
    transaction_isolation
  -----------------------
   read committed
  (1 row)
  
  postgres=#


##  3.	начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');

    postgres=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
    INSERT 0 1
    postgres=*#


##  4.	сделать select from persons во второй сессии

  postgres=# select * from persons;
   id | first_name | second_name
  ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
  (2 rows)


##  5.	видите ли вы новую запись и если да то почему?

нет, так как уровень изоляции read committed, а commit - а не было

##  6.	завершить первую транзакцию - commit;

    postgres=*# commit;
    COMMIT
    postgres=#


##  7.	сделать select from persons во второй сессии

  postgres=# select * from persons;
   id | first_name | second_name
  ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    3 | sergey     | sergeev
  (3 rows)
  
  postgres=#

##  8.	видите ли вы новую запись и если да то почему?

да, так как выполнен commit;


##  9.	завершите транзакцию во второй сессии

  postgres=# commit;
  WARNING:  there is no transaction in progress
  COMMIT
  postgres=#

##  10. начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;

  postgres=*# set transaction isolation level repeatable read;
  SET
  postgres=*# show transaction isolation level;
   transaction_isolation
  -----------------------
   repeatable read
  (1 row)
  
  postgres=*#


##  11.	в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');

  postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
  INSERT 0 1
  postgres=*#

##  11.	сделать select* from persons во второй сессии*

  postgres=# select * from persons;
   id | first_name | second_name
  ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    3 | sergey     | sergeev
  
  (3 rows)
  
  postgres=# ^C



##  12.	видите ли вы новую запись и если да то почему?

нет, так как для данной транзакции был установлен isolation level repeatable read


##  11.	завершить первую транзакцию - commit;

  postgres=*# commit;
  COMMIT
  postgres=#

##  12.	сделать select from persons во второй сессии

  postgres=# select * from persons;
   id | first_name | second_name
  ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    3 | sergey     | sergeev
    4 | sveta      | svetova
  (4 rows)
  
  postgres=# ^C

##  13.	видите ли вы новую запись и если да то почему?

да, так как прошёл commit
