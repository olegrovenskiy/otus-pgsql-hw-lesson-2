# Инсталляция PostgreeSQL

1.    поставить PostgreSQL
2.    зайти вторым ssh (вторая сессия)
3.    запустить везде psql из под пользователя postgres

Инсталяция PostgreSQL-15 на ВМ в виртуализации ВМваре, Centos-7, как корпоративный стандарт


1. Centos 7 10.102.6.27

2. yum update

Использовал информацию с :

https://www.postgresql.org/download/linux/redhat/
https://orcacore.com/install-postgresql-15-centos-7/

3. yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

4. yum install -y postgresql15-server

получил ошибку

           Error: Package: postgresql15-15.5-1PGDG.rhel7.x86_64 (pgdg15)
                      Requires: libzstd >= 1.4.0

для решения нашёл соответствующий пакет на https://pkgs.org/ (нужный пакет в соответствии с архитектурой)

6. yum localinstall https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libzstd-1.5.5-1.el7.x86_64.rpm


7. yum install -y postgresql15-server

                      Installed:
                        postgresql15-server.x86_64 0:15.5-1PGDG.rhel7
                      
                      Dependency Installed:
                        libicu.x86_64 0:50.2-4.el7_7                             postgresql15.x86_64 0:15.5-1PGDG.rhel7
                        postgresql15-libs.x86_64 0:15.5-1PGDG.rhel7


8. psql -V

                              [root@mck-network-test ~]#  psql -V
                              psql (PostgreSQL) 15.5
                              [root@mck-network-test ~]#


9.

        [root@mck-network-test ~]# postgresql-15-setup initdb
        Initializing database ... OK
        
        [root@mck-network-test ~]#
        [root@mck-network-test ~]#
        [root@mck-network-test ~]# systemctl enable postgresql-15
        Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-15.service to /usr/lib/systemd/system/postgresql-15.service.
        [root@mck-network-test ~]# systemctl start postgresql-15
        [root@mck-network-test ~]# systemctl status postgresql-15
        ● postgresql-15.service - PostgreSQL 15 database server
           Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
           Active: active (running) since Fri 2023-12-01 03:09:52 EST; 9s ago
             Docs: https://www.postgresql.org/docs/15/static/
          Process: 24300 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
         Main PID: 24306 (postmaster)
           CGroup: /system.slice/postgresql-15.service
                   ├─24306 /usr/pgsql-15/bin/postmaster -D /var/lib/pgsql/15/data/
                   ├─24308 postgres: logger
                   ├─24309 postgres: checkpointer
                   ├─24310 postgres: background writer
                   ├─24312 postgres: walwriter
                   ├─24313 postgres: autovacuum launcher
                   └─24314 postgres: logical replication launcher
        
        Dec 01 03:09:52 mck-network-test.mgc.local systemd[1]: Starting PostgreSQL 15 database server...
        Dec 01 03:09:52 mck-network-test.mgc.local postmaster[24306]: 2023-12-01 03:09:52.256 EST [24306] LOG:  re...ss
        Dec 01 03:09:52 mck-network-test.mgc.local postmaster[24306]: 2023-12-01 03:09:52.256 EST [24306] HINT:  F...".
        Dec 01 03:09:52 mck-network-test.mgc.local systemd[1]: Started PostgreSQL 15 database server.
        Hint: Some lines were ellipsized, use -l to show in full.
        [root@mck-network-test ~]#




открытие 5432 для внешних пользователей

https://www.dmosk.ru/miniinstruktions.php?mini=pgsql-remote&ysclid=lpmjvsqk9g399750990
+
firewall
https://www.server-world.info/en/note?os=CentOS_Stream_9&p=postgresql15&f=2


To change the PostgreSQL user's password, follow these steps:

log in into the psql console:

sudo -u postgres psql
Then in the psql console, change the password and quit:

postgres=# \password postgres
Enter new password: <new-password>
postgres=# \q






# Home Work otus-pgsql-hw-lesson-2

зайти вторым ssh (вторая сессия)
запустить везде psql из под пользователя postgres

done


выключить auto commit

    \set AUTOCOMMIT off

done

сделать:

##  1.	в первой сессии новую таблицу и наполнить ее данными 

create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

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
