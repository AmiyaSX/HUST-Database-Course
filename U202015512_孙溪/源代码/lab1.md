1. 创建数据库
```PostgreSQL
create database beijing2022;
```
2. 创建表及表的主码约束
```PostgreSQL
create table t_emp(
    id INT PRIMARY KEY,
    name VARCHAR(32),
    deptld INT,
    salary FLOAT
);
```
3. 创建外码约束(foreign key)
```PostgreSQL
create table dept(
    deptNo int primary key,
    deptName varchar(32)
);
create table staff(
    staffNo int primary key,
    staffName varchar(32),
    gender char(1),
    dob date,
    salary numeric(8,2),
    deptNo int,
    CONSTRAINT FK_staff_deptNo foreign key(deptNo) references dept(deptNo)
);
```
4. CHECK约束
```PostgreSQL
create table products(
    pid char(10) primary key,
    name varchar(32),
    brand char(10),
    price int,
    constraint CK_products_brand check(brand in ('A','B')),
    constraint CK_products_price check(price > 0)
);
```
5. DEFAULT约束
```PostgreSQL
create table hr(
    id char(10) primary key,
    name varchar(32) not null,
    mz char(16) default '汉族'
);
```
6. UNIQUE约束
``` PostgreSQL
create table s(
    sno char(10),
	name varchar(32) NOT NULL,
	ID char(18) UNIQUE,
	primary key(sno)
);
```
