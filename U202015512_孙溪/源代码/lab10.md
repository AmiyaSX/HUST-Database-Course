1. 不可重复读
```postgresql
-- 事务1:
-- 请设置适当的事务隔离级别

set session transaction isolation level read COMMITTED;

-- 开启事务
start transaction;
-- 时刻1 - 事务1读航班余票:
insert into result(t,tickets) 
select 1 t, tickets from ticket where flight_no = 'CZ5525';
-- 添加等待代码，确保事务2的第一次读取在事务1修改前发生

SELECT pg_sleep(2);

-- 时刻3 - 事务1修改余票，并立即读取:
update ticket set tickets = tickets - 1 where flight_no = 'CZ5525';
insert into result(t,tickets)
select 1 t, tickets from ticket where flight_no = 'CZ5525';

-- SELECT pg_sleep(1);
commit;

-- 时刻6 - 事务1在t2也提交后读取余票
-- 添加代码，确保事务1在事务2提交后读取
SELECT pg_sleep(3);
insert into result(t,tickets)
select 1 t, tickets from ticket where flight_no = 'CZ5525';
```

2. 幻读
```postgresql
-- 事务1（采用事务隔离级别- read committed）:
set session transaction isolation level repeatable read;

-- 开启事务
start transaction;

-- 第1次查询余票超过300张的航班信息
insert into result
select *,1 t from ticket where tickets > 300;
select pg_sleep(4);
-- 修改航班MU5111的执飞机型为A330-300：
update ticket set aircraft = 'A330-200' where flight_no = 'MU5111'; 
-- 第2次查询余票超过300张的航班信息

insert into result
select *,2 t from ticket where tickets > 300;
commit;
```

3. 主动加锁保证可重复读
```postgresql
-- 事务1: 
select  pg_sleep(1);
set session transaction isolation level read committed;
start transaction;
-- 第1次查询航班'MU2455'的余票
select tickets from ticket where flight_no = 'MU2455' for update;
select pg_sleep(5);
-- 第2次查询航班'MU2455'的余票
select tickets from ticket where flight_no = 'MU2455' for update;
commit;
-- 第3次查询所有航班的余票，发生在事务2提交后
select pg_sleep(1);
select * from ticket  order by flight_no;
```
